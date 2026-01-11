---
layout: post
title: "Shader Hot Reloading"
perma-link: "/havtorn/shader_hot_reload/"
featured-image: /assets/images/shaderHotCompile.gif
tags: [havtorn, rendering, shader]
---

<!--excerpt-begin-->
Reloading shaders while the engine is running vastly decreases the development and debugging times of graphics features. Granted we're 
talking about explicitly named, persistent shaders here and not some material-based, automatically generated solution. As you're 
starting out making an engine this is most likely the setup you'll be using for a while, and while implementing features such as 
<a href="https://en.wikipedia.org/wiki/Screen_space_ambient_occlusion">SSAO</a> or <a href="https://en.wikipedia.org/wiki/Tone_mapping">tone mapping</a>,
 it will be incredibly helpful to be able to tweak parameters or formulas on the fly.
<!--excerpt-end-->

## File Watching

Havtorn has a simple, request-based file watching system running on a separate thread, which is also used to <a href="{{site.baseurl}}/asset-hot-reloading/">hot reload assets</a>. 
We set it up to run on a separate thread on initialization, 

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++
CFileWatcher::~CFileWatcher()
{
    ShouldEndThread = true;
}

bool CFileWatcher::Init(CThreadManager* threadManager)
{
    if (!threadManager)
        return false;

    threadManager->PushJob(std::bind(&CFileWatcher::UpdateChanges, this));
    return true;
}

void CFileWatcher::UpdateChanges()
{
    while (!ShouldEndThread)
    {	
        {
            std::lock_guard<std::mutex> lock(Mutex);
            for (const auto& [path, currentTimestamp] : WatchedFiles)
            {
                const U64 latestTimeStamp = GetFileTimestamp(path);
                if (latestTimeStamp > currentTimestamp)
                {
                    QueuedFileChanges.push(path);
                    WatchedFiles[path] = latestTimeStamp;
                }
            }
        }

        std::this_thread::sleep_for(std::chrono::milliseconds(SleepDurationMilliseconds));
    }
}
```
</details>
{::options parse_block_html="false" /}

The `WatchedFiles` property is a `std::map` we write to when requesting a file watch on the main thread. When we do, we provide an `std::function` object, 
which we convert to an `std::shared_ptr` when requesting removal (to make sure we remove the correct entry). In our case the file watcher 
is used only by big systems such as the editor and the renderer, which have longer lifetimes, so this isn't a huge issue. Though it would make sense to associate 
the callbacks with a handle for direct access, and to validate the objects the callbacks are associated with prior to calling them, if needed.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++
bool CFileWatcher::WatchFileChange(const std::string& filePath, CFileChangeCallback callback)
{
    const std::filesystem::path newPath = filePath.c_str();

    if (!std::filesystem::exists(newPath))
        return false;

    std::lock_guard<std::mutex> lock(Mutex);
    StoredCallbacks[newPath].push_back(callback);

    if (!WatchedFiles.contains(newPath))
        WatchedFiles.emplace(newPath, GetFileTimestamp(newPath));

    return true;
}

void CFileWatcher::StopWatchFileChange(const std::string& filePath, CFileChangeCallback callback)
{	
    const std::filesystem::path existingPath = filePath.c_str();

    if (!std::filesystem::exists(existingPath))
        return;

    if (!StoredCallbacks.contains(existingPath))
        return;

    std::lock_guard<std::mutex> lock(Mutex);
    std::vector<CFileChangeCallback>& callbackContainer = StoredCallbacks.at(existingPath);
    ...
    // Find callback to erase by some means
    ...
    callbackContainer.erase(it);

    if (!callbackContainer.empty())
        return;

    StoredCallbacks.erase(existingPath);
    WatchedFiles.erase(existingPath);
}
```
</details>
{::options parse_block_html="false" /}

While `UpdateChanges` runs on the file watch thread, we try to `FlushChanges` at a set point on the main thread, where we go through
the queued up changes from the file watch thread and call the callbacks. 

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++
void CFileWatcher::FlushChanges()
{
    std::lock_guard<std::mutex> lock(Mutex);
    while (!QueuedFileChanges.empty())
    {
        const std::filesystem::path& filePath = QueuedFileChanges.front();
        if (!StoredCallbacks.contains(filePath))
        {
            QueuedFileChanges.pop();
            continue;
        }

        const std::vector<CFileChangeCallback>& callbacks = StoredCallbacks[filePath];
        for (const CFileChangeCallback& callback : callbacks)
        callback(filePath.string());
		
        QueuedFileChanges.pop();
    }
}
```
</details>
{::options parse_block_html="false" /}

## Shader Hot Reload

At the point of loading shaders, we also find the corresponding source files and start watching them for changes. Naturally, we wouldn't want 
or need to do this for release builds. Notably I'm making it easy for myself here - again because we have an explicit, static set of shaders -
by storing the shaders in multiple `std::array`s, so that we can just switch them out when reloading them.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++
std::string CRenderStateManager::AddShader(const std::string& filePath, const U64 index, const EShaderType shaderType)
{
    ...

    switch (shaderType)
    {
    case EShaderType::Vertex:
    {
        ...
    }
    ...
    case EShaderType::Pixel:
    {
        // Index directly into the std::array

        if (PixelShaders[index] != nullptr)
            PixelShaders[index]->Release();

        CPixelShader* newPixelShader = nullptr;
        UGraphicsUtils::CreatePixelShader(filePath, Framework, &newPixelShader);
        PixelShaders[index] = pixelShader;
    }
    break;
    }

    const std::string sourceFile = UGeneralUtils::DeriveSourceFileFromPath(filePath, "hlsl");
    if (!ShaderInitData.contains(sourceFile))
    {
        GEngine::GetFileWatcher()->WatchFileChange(sourceFile, std::bind(&CRenderStateManager::OnShaderSourceChange, this, std::placeholders::_1));
        
        // NW: Save some extra context about the file so we can easily replace it later
        ShaderInitData.emplace(sourceFile, SShaderInitData{ filePath, shaderType, index });
    }

    ...
}
```
</details>
{::options parse_block_html="false" /}

When the source file changes, we queue up the file path to be recompiled to a new binary at a good time, similar to what we did in the `FileWatcher`. 
In this case, we go flush the changes when the main thread and render thread sync and swap resources. 

This code is specific to DirectX11, but the same principles apply. Implementations and compilers used for Vulkan and even DirectX12 will differ, 
but the information is pretty easy to find. Havtorn doesn't yet support the newer generation of backends so I will just show our solution for the 
DirectX11 case here. 

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++
void CRenderStateManager::OnShaderSourceChange(const std::string& filePath)
{
    std::lock_guard<std::mutex> lock(ShaderRecompileMutex);
    QueuedShaderRecompiles.push(filePath);
}

void CRenderStateManager::FlushShaderChanges()
{
    // NW: Use DXC for DirectX12 Shader Model 6.0 and above, or one of the Vulkan shader compilers to compile into SPIR-V for Vulkan, e.g. glslc or glslang
    // https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-part1
    // https://github.com/KhronosGroup/glslang

    std::lock_guard<std::mutex> lock(ShaderRecompileMutex);
    while (!QueuedShaderRecompiles.empty())
    {
        const std::string changedSourceFile = QueuedShaderRecompiles.front();

        const SShaderInitData initData = ShaderInitData.at(changedSourceFile);
        const std::wstring wideSourceFilePath = { changedSourceFile.begin(), changedSourceFile.end() };
        const std::wstring wideOutputFilePath = { initData.OutputFileName.begin(), initData.OutputFileName.end() };

        ID3DBlob* compiledContents = nullptr;
        ID3DBlob* errorMessages = nullptr;

        std::string shaderModel;
        switch (initData.ShaderType)
        {
        case EShaderType::Pixel:
            shaderModel = "ps_5_0";
            break;
        case EShaderType::Geometry:
            shaderModel = "gs_5_0";
            break;
        case EShaderType::Compute:
            shaderModel = "cs_5_0";
            break;
        case EShaderType::Vertex:
            [[fallthrough]];
        default:
            shaderModel = "vs_5_0";
        }

        UShaderIncludeHandler customIncludeHandler;
        const HRESULT compileResult = D3DCompileFromFile(wideSourceFilePath.c_str(), nullptr, &customIncludeHandler, "main", shaderModel.c_str(), 0, 0, &compiledContents, &errorMessages);
        if (compileResult != S_OK)
        {
            HV_LOG_ERROR("CRenderStateManager::OnShaderSourceChange: Shader %s could not be recompiled: %s", changedSourceFile.c_str(), (char*)errorMessages->GetBufferPointer());
            QueuedShaderRecompiles.pop();
            errorMessages->Release();
            break;
        }

        const HRESULT rewriteResult = D3DWriteBlobToFile(compiledContents, wideOutputFilePath.c_str(), TRUE);
        if (rewriteResult != S_OK)
        {
            HV_LOG_ERROR("CRenderStateManager::OnShaderSourceChange: Shader %s was successfully recompiled, but output file could not be overwritten.", changedSourceFile.c_str());
            QueuedShaderRecompiles.pop();
            compiledContents->Release();
            break;
        }

        compiledContents->Release();

        // NW: Re-add shader using the context we saved before
        AddShader(initData.OutputFileName, initData.ShaderIndex, initData.ShaderType);

        HV_LOG_INFO("Shader source file %s was recompiled.", changedSourceFile.c_str());

        QueuedShaderRecompiles.pop();
    }
}
```
</details>
{::options parse_block_html="false" /}

Note the custom *include handler* used in the compilation call. DirectX (I'm not sure about Vulkan) needs this to know what to do when it comes across an `#include` 
in the hlsl source during compilation. You can provide a default one by using the `D3D_COMPILE_STANDARD_FILE_INCLUDE` macro, which will find files relative to the 
source file directory, or pass `nullptr` if you don't include any files in the shader source. In our case, we're including files from a specific `Includes` directory 
in the shader source directory, and some include files include other include files also relative to this directory. I ended up with this custom include handler for use 
under these very specific conditions.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++
class UShaderIncludeHandler : public ID3DInclude
{
    HRESULT Open(D3D_INCLUDE_TYPE /*includeType*/, LPCSTR pFileName, LPCVOID /*pParentData*/, LPCVOID* ppData, UINT* pBytes) override
    {
        // NW: Only include files in the Shaders/Includes folder in shaders.
        const std::string shaderIncludeSource = UGeneralUtils::ExtractParentDirectoryFromPath(UFileSystem::GetWorkingPath()) + "Source/Engine/Graphics/Shaders/Includes/";
        const std::string inputFileName = UGeneralUtils::ExtractFileNameFromPath(pFileName);
        const std::string filePath = shaderIncludeSource + inputFileName;

        if (!UFileSystem::Exists(filePath))
            return E_FAIL;

        U32 fileSize = STATIC_U32(UFileSystem::GetFileSize(filePath));
        char* data = new char[fileSize];
        
        std::ifstream inputStream;
        inputStream.open(filePath.c_str(), fstream::in | fstream::binary);
        
        inputStream.read(data, fileSize);
        inputStream.close();

        *pBytes = fileSize;
        *ppData = data;
            
        return S_OK;
    }

    HRESULT Close(LPCVOID pData) override
    {
        delete[] pData;
        return S_OK;
    }
};
```
</details>
{::options parse_block_html="false" /}