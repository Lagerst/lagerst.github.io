## Welcome

## [return to index](./index.html)

### [Cross Platform] Node-Addon-Api: C++ addon for node.js

To run native C++ code in node.js, you can use [node-addon-api](https://github.com/nodejs/node-addon-api). [Examples](https://github.com/nodejs/node-addon-examples) are given here. Also a [desktop camera](https://github.com/Lagerst/DesktopCamera) is implemented with Electron and opencv in my github.

```cpp
// sample.cc
#include <napi.h>

Napi::String Method(const Napi::CallbackInfo& info) {
  Napi::Env env = info.Env();
  return Napi::String::New(env, "world");
}

Napi::Object Init(Napi::Env env, Napi::Object exports) {
  exports.Set(Napi::String::New(env, "hello"),
              Napi::Function::New(env, Method));
  return exports;
}

NODE_API_MODULE(hello, Init)
```
```js
// sample.js
var addon = require('bindings')('hello');
console.log(addon.hello()); // 'world'

// package.json -
// binding.gyp -
```

Advanced Usage:
If you want to invoke a async callback for current node.js loop from current or another thread, you can wrap a **async_task** based on [libuv](http://libuv.org/).

### [Cross Platform] Google Crashpad

Crashpad is a library for capturing, storing and transmitting postmortem crash reports from a client to an upstream collection server. Crashpad aims to make it possible for clients to capture process state at the time of crash with the best possible fidelity and coverage, with the minimum of fuss.

Crashpad also provides a facility for clients to capture dumps of process state on-demand for diagnostic purposes.

Crashpad additionally provides minimal facilities for clients to adorn their crashes with application-specific metadata in the form of per-process key/value pairs. More sophisticated clients are able to adorn crash reports further through extensibility points that allow the embedder to augment the crash report with application-specific metadata.

[Overview Design](https://chromium.googlesource.com/crashpad/crashpad/+/master/doc/overview_design.md)

[Source Code](https://chromium.googlesource.com/crashpad/crashpad/)

### [Cross Platform] Launch a Privileged Task

**MacOS**

Use [STPrivilegedTask](https://github.com/sveinbjornt/STPrivilegedTask) which is deprecated but can still be used, or [SMJobBless](https://developer.apple.com/library/archive/samplecode/EvenBetterAuthorizationSample/Introduction/Intro.html) which is complicated but less likely to be deprecated in the future. Or use [base/mac/authorization_util.h](https://source.chromium.org/chromium/chromium/src/+/main:base/mac/authorization_util.h) which also use deprecated methods. 
> AuthorizationExecuteWithPrivileges is deprecated in macOS 10.7, but no good replacement exists. https://crbug.com/593133. 

To custom text or icon on poped dialog in ***STPrivilegedTask***, call
```obj-c
NSString* dialogText = @"Needs permission to upgrade";
AuthorizationEnvironment customAuthorizationEnvironment;
customAuthorizationEnvironment.items = kAuthEnv;
customAuthorizationEnvironment.count = 1;

// The OS will dispay |prompt| along with a sentence asking the user to type
// the "password to allow this."
const char* prompt_c = [dialogText UTF8String];
size_t prompt_length = prompt_c ? strlen(prompt_c) : 0;

kAuthEnv[0].name = kAuthorizationEnvironmentPrompt;
kAuthEnv[0].valueLength = prompt_length;
kAuthEnv[0].value = (void *)prompt_c;
kAuthEnv[0].flags = 0;
```
where the value should be a localized UTF-8 string. See [AuthorizationEnvironment](https://developer.apple.com/documentation/security/authorizationenvironment?language=objc) and [kAuthorizationEnvironmentPrompt](https://developer.apple.com/documentation/security/kauthorizationenvironmentprompt?changes=_8&language=objc).

Special Notes:
On posix system files created by privieged task via base::File::FLAG_CREATE in chromium/src/base/file.h will be locked for only root user to access. The file is created with default mode `int mode = S_IRUSR | S_IWUSR;`, which only allow the current user to read and write. Unfortunately, the current user is root, and when normal user launch the app it cannot access the file and might never be able to read/write the file, which is fatal to some settings stored on local disk. So avoid such operation in privileged tasks. Meanwhile some files may be overwriten by deleting & recreating, so they will also face the problem.

**Windows**

See [base/process/launch_win.cc](https://chromium.googlesource.com/chromium/src/base/+/master/process/launch_win.cc) in chromium & [Launching Applications](https://docs.microsoft.com/en-us/windows/win32/shell/launch) at Microsoft Docs.

```cpp
SHELLEXECUTEINFO shex_info = {};
shex_info.cbSize = sizeof(shex_info);
shex_info.fMask = SEE_MASK_NOCLOSEPROCESS;
shex_info.hwnd = GetActiveWindow();
shex_info.lpVerb = L"runas";
shex_info.lpFile = file.c_str();
shex_info.lpParameters = arguments.c_str();
shex_info.lpDirectory = nullptr;
shex_info.nShow = options.start_hidden ? SW_HIDE : SW_SHOWNORMAL;
shex_info.hInstApp = nullptr;

ShellExecuteEx(&ShExecInfo);
WaitForSingleObject(ShExecInfo.hProcess,INFINITE);
```

### [Cross Platform] Chromium Mojo IPC

**[Mojo IPC](https://chromium.googlesource.com/chromium/src/+/master/mojo/README.md)**

**Embedding Mojo**

```cpp
mojo::core::Init();
base::Thread ipc_thread("ipc!");
ipc_thread.StartWithOptions(
    base::Thread::Options(base::MessagePumpType::IO, 0));
mojo::core::ScopedIPCSupport ipc_support(
    ipc_thread.task_runner(),
    mojo::core::ScopedIPCSupport::ShutdownPolicy::CLEAN);
```

**[Using Named Pipe](https://source.chromium.org/chromium/chromium/src/+/main:mojo/public/cpp/platform/README.md)**

A NamedPlatformChannel upon construction will begin listening on a platform-specific primitive (a named pipe server on Windows, a domain socket server on POSIX, etc.). The globally reachable name of the server (e.g. the socket path) can be specified at construction time via NamedPlatformChannel::Options::server_name, but if no name is given, a suitably random one is generated and used.
```cpp
// Sending/Receiving Invitiaion.
{
  // In one process
  mojo::NamedPlatformChannel::Options options;
  mojo::NamedPlatformChannel named_channel(options);
  OutgoingInvitation::Send(std::move(invitation),
                           named_channel.TakeServerEndpoint());
  SendServerNameToRemoteProcessSomehow(named_channel.GetServerName());

  // In the other process
  void OnGotServerName(const mojo::NamedPlatformChannel::ServerName& name) {
    // Connect to the server.
    mojo::PlatformChannelEndpoint endpoint =
        mojo::NamedPlatformChannel::ConnectToServer(name);

    // Proceed normally with invitation acceptance.
    auto invitation = mojo::IncomingInvitation::Accept(std::move(endpoint));
    // ...
  }
}
// Sending/Receiving Message.
{
  // Optional: Wait till WRITABLE or PEER_CLOSED
  MojoHandleSignalsState state;
  mojo::Wait(pipe_handle_,
             MOJO_HANDLE_SIGNAL_WRITABLE | MOJO_HANDLE_SIGNAL_PEER_CLOSED,
             &state);
  // MOJO_HANDLE_SIGNAL_READABLE | MOJO_HANDLE_SIGNAL_PEER_CLOSED (IF READ)
  if (state.satisfied_signals &
      MOJO_HANDLE_SIGNAL_WRITABLE) /* MOJO_HANDLE_SIGNAL_READABLE */ {
    MojoResult ret =
        mojo::WriteMessageRaw(pipe_handle_, message.c_str(), message.size(),
                              nullptr, 0, MOJO_WRITE_MESSAGE_FLAG_NONE);
    /*  std::vector<uint8_t> payload;
        MojoResult ret = mojo::ReadMessageRaw(pipe_handle_, &payload, nullptr,
                                              MOJO_READ_MESSAGE_FLAG_NONE);*/
  } else { /* ... */}
}
```

### [Windows] MSVC Compiler Options /MD, /MT

[(Use Run-Time Library)](https://docs.microsoft.com/en-us/cpp/build/reference/md-mt-ld-use-run-time-library?view=msvc-160) Indicates whether a multithreaded module is a DLL and specifies retail or debug versions of the run-time library.

/MD\[d\] causes the application to use the multithread-specific and DLL-specific version of the run-time library.

/MT\[d\] causes the application to use the multithread, static version of the run-time library.

Each time a new process attempts to use the DLL, the operating system creates a separate copy of the DLL's data: this is called "process attach". The run-time library code for the DLL calls the constructors for all the global objects, if any, and then calls the DllMain function with process attach selected. The opposite situation is process detach: the run-time library code calls DllMain with process detach selected and then calls a list of termination functions including atexit functions, destructors for the global objects, and destructors for the static objects. Note that the order of events in process attach is the reverse of that in process detach.
The run-time library code is also called during thread attach and thread detach, but the run-time code does no initialization or termination on its own.

Linking with MD has advantages:

1. Different modules (i.e. DLLs or EXEs) can exchange "memory ownership". For example, a memory allocated through new/malloc in one module can be reallocated/deleted/freed by another. This is very helpful if you use STL in the interface between your modules.
2. The runtime has some "global data". Linking with MT means that this "global data" will not be shared, while with MD, it will be shared. This means that with MD, you can pass some data from one module to another without risk.
3. your executable can be smaller (since it doesn't have the library embedded in it), and I believe that at very least the code segment of a DLL is shared amongst all processes that are actively using it (reducing the total amount of RAM consumed).

Linking with MD has disadvantages:

1. while a module compiled with MD will link with a DLL at the moment of execution. if DLL will not found in the machine then your application will be crashed.

Linking with MT has advantages:

1. If you use /MT, your executable won't depend on a DLL being present on the target system. If you're wrapping this in an installer, it probably won't be an issue and you can go either way.

To set this compiler option in the Visual Studio development environment
1. Open the project's Property Pages dialog box. For details, see Set C++ compiler and build properties in Visual Studio.
2. Select the Configuration Properties > C/C++ > Code Generation property page.
3. Modify the Runtime Library property.

Notes:

1. Anyway, a release and debug versions of the same module should link with the same category of runtime. If your release links with MT, then your debug should link with MTd. In the same way, if your release links with MD, your debug should link with MD\[d\].

2. Now, you should avoid mixing in the same process different modules linked with different run times (Note that the release and debug run times are different run times). violating this rule could lead to mysterious crashes. Potential errors are given [here](<https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2012/ms235460(v=vs.110)?redirectedfrom=MSDN>). When you pass C Run-time (CRT) objects such as file handles, locales, and environment variables into or out of a DLL (function calls across the DLL boundary), unexpected behavior can occur if the DLL, as well as the files calling into the DLL, use different copies of the CRT libraries. A related problem can occur when you allocate memory (either explicitly with new or malloc, or implicitly with strdup, strstreambuf::str, and so on) and then pass a pointer across a DLL boundary to be freed. This can cause a memory access violation or heap corruption if the DLL and its users use different copies of the CRT libraries. Another symptom of this problem can be an error in the output window during debugging such as: `HEAP[]: Invalid Address specified to RtlValidateHeap(#,#)`.

Tips:

`dumpbin /directives target.lib` can be used to tell if a lib was compiled with /mt or /md:

1. /DEFAULTLIB:MSVCRT (module compiled with /MD)
2. /DEFAULTLIB:MSVCRTD (module compiled with /MDd)
3. /DEFAULTLIB:LIBCMT (module compiled with /MT)
4. /DEFAULTLIB:LIBCMTD (module compiled with /MTd)

### [Windows] AppUserModelID

The application id (System.AppUserModel.ID property) of a Windows 7 shortcut.

In Windows 7 (or above), taskbar items are grouped by a string known as the application id or AppId. This can be set in the shortcut that launches a program, or by the application itself. See [AppUserModelIDs](http://msdn.microsoft.com/en-us/library/dd378459%28VS.85%29.aspx) for more information. There's a [tool](https://code.google.com/archive/p/win7appid/) to modify the AppUserModelID, from where we can learn to modify it in our own code.

```cpp
EXTERN_C const PROPERTYKEY DECLSPEC_SELECTANY PKEY_AppUserModel_ID = {
    {0x9F4C2855, 0x9F79, 0x4B39, {
     0xA8, 0xD0, 0xE1, 0xD4, 0x2D, 0xE1, 0xD5, 0xF3}}, 5};

// Need addtional fault tolerance mechanism.
// Do Initialize;
CoInitializeEx(NULL, COINIT_APARTMENTTHREADED);
IShellLink* link;
CoCreateInstance(CLSID_ShellLink, NULL, CLSCTX_INPROC_SERVER,
                 IID_PPV_ARGS(&link));
link->QueryInterface(IID_PPV_ARGS(&file));
file->Load(argv[1], STGM_READWRITE);
IPropertyStore* store;
link->QueryInterface(IID_PPV_ARGS(&store));
// Get AppUserModelID;
PROPVARIANT pv;
store->GetValue(PKEY_AppUserModel_ID, &pv);
if (pv.vt != VT_EMPTY) {
  if (pv.vt != VT_LPWSTR) {
    _Error("Unexpected property value type = ", pv.vt);
  }
  _CurrentAppId(pv.pwszVal)
} else {
  _NoCurrentAppId();
}
PropVariantClear(&pv);
// Set AppUserModelID;
pv.vt = VT_LPWSTR;
pv.pwszVal = "new";
store->SetValue(PKEY_AppUserModel_ID, pv);
pv.pwszVal = NULL;
PropVariantClear(&pv);
store->Commit();
file->Save(NULL, TRUE);
// Release;
store->Release();
file->Release();
link->Release();
```

### [Windows] Dynamic-Link Library Security

**See [Dynamic-Link Library Security](https://docs.microsoft.com/zh-cn/windows/win32/dlls/dynamic-link-library-security)**

When an application dynamically loads a dynamic-link library without specifying a fully qualified path name, Windows attempts to locate the DLL by searching a well-defined set of directories in a particular order, as described in Dynamic-Link Library Search Order. If an attacker gains control of one of the directories on the DLL search path, it can place a malicious copy of the DLL in that directory. This is sometimes called a DLL preloading attack or a binary planting attack. If the system does not find a legitimate copy of the DLL before it searches the compromised directory, it loads the malicious DLL. If the application is running with administrator privileges, the attacker may succeed in local privilege elevation.

For example, suppose an application is designed to load a DLL from the user's current directory and fail gracefully if the DLL is not found. The application calls LoadLibrary with just the name of the DLL, which causes the system to search for the DLL. Assuming safe DLL search mode is enabled and the application is not using an alternate search order, the system searches directories in the following order:
1. The directory from which the application loaded.
2. The system directory.
3. The 16-bit system directory.
4. The Windows directory.
5. The current directory.
6. The directories that are listed in the PATH environment variable.
Continuing the example, an attacker with knowledge of the application gains control of the current directory and places a malicious copy of the DLL in that directory. When the application issues the LoadLibrary call, the system searches for the DLL, finds the malicious copy of the DLL in the current directory, and loads it. The malicious copy of the DLL then runs within the application and gains the privileges of the user.

**common ways to avoid this risk**
1. Call [SetDefaultDllDirectories](https://docs.microsoft.com/zh-cn/windows/win32/api/libloaderapi/nf-libloaderapi-setdefaultdlldirectories) and use [/DELAYLOAD](https://docs.microsoft.com/en-us/cpp/build/reference/delayload-delay-load-import?view=msvc-160).
2. Some dll is loaded before main entry. Do not call them directly, instead we can call [GetProcAddress](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress) to load the function from a dynamicly loaded DLL module handle.

**tools to examine DLL load operations in your application**
[Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)

### [MacOS] com.apple.quarantine and App Translocation

In macOS Sierra, Apple added a strange security feature called **App Translocation** (sometimes known as Gatekeeper Path Randomization) which means that after downloading an application, if you do not move the resulting application somewhere (anywhere!), with the Finder (you must use the Finder!), the application will be run as if it is located at a randomly chosen path by the system. One of the consequence of this is that the version updates will fail (because the application cannot replace itself).

**com.apple.quarantine** is an [extended attribute](https://en.wikipedia.org/wiki/Extended_file_attributes#Linux) which is added to a file if downloaded from the Internet. The quarantine attribute is added by the OS so that it can ask for user confirmation the first time the downloaded program is run. This attribute might also be added if the file created by a process which has this attribute.

But note that not all app with com.apple.quarantine will run as translocated. Move the app with Finder will cause the app run normally. The policy that whether the app should run translocated, if the application is running translocated, how to check where the original path is and some more are implemented in [Security/SecTranslocate.h](https://opensource.apple.com/source/Security/Security-57740.51.3/OSX/libsecurity_translocate/lib/SecTranslocate.h.auto.html).

```obj-c
Boolean SecTranslocateIsTranslocatedURL(CFURLRef path, bool* isTranslocated, CFErrorRef* __nullable error)
__OSX_AVAILABLE(10.12);

Boolean SecTranslocateURLShouldRunTranslocated(CFURLRef path, bool* shouldTranslocate, CFErrorRef* __nullable error)
__OSX_AVAILABLE(10.12);

CFURLRef __nullable SecTranslocateCreateOriginalPathForURL(CFURLRef translocatedPath, CFErrorRef* __nullable error)
__OSX_AVAILABLE(10.12);
```

The translocation policy descripted in annotation as followed:

1. If path is already on a nullfs mountpoint - no translocation
2. No quarantine attributes - no translocation
3. If QTN_FLAG_DO_NOT_TRANSLOCATE is set or QTN_FLAG_TRANSLOCATE is not set - no translocations
4. Otherwise, if QTN_FLAG_TRANSLOCATE is set - translocation

You can avoid App Translocation by breaking any one of the necessary conditions above. For example, remove the attribute of com.apple.quarantine. Run `xattr path/to/target` to watch all of the extended attributes and run `xattr -d -r com.apple.quarantine path/to/target` to remove the quarantine attributes.

### [MacOS] Set Default About Panel

Take electron's implementation as a reference.

```obj-c
base::DictionaryValue about_panel_options_ = {};
// about_panel_options_ is a base::DictionaryValue with
//    applicationName String (optional) - The app's name.
//    applicationVersion String (optional) - The app's version.
//    copyright String (optional) - Copyright information.
//    version String (optional) macOS - The app's build version number.
//    credits String (optional) macOS Windows - Credit information.
//    authors String[] (optional) Linux - List of app authors.
//    website String (optional) Linux - The app's website.
//    iconPath String (optional) Linux Windows - Path to the app's icon in a JPEG or PNG file format. On Linux, will be shown as 64x64 pixels while retaining aspect ratio.

NSDictionary* options = DictionaryValueToNSDictionary(about_panel_options_);

// Credits must be a NSAttributedString instead of NSString
id credits = options[@"Credits"];
if (credits != nil) {
  NSMutableDictionary* mutable_options = [options mutableCopy];
  NSDictionary* styles =
      @{NSFontAttributeName : [NSFont systemFontOfSize:10]};
  mutable_options[@"Credits"] =
      [[[NSAttributedString alloc] initWithString:(NSString*)credits
                                        attributes:styles] autorelease];
  options = [NSDictionary dictionaryWithDictionary:mutable_options];
}

[[AtomApplication sharedApplication]
    orderFrontStandardAboutPanelWithOptions:options];
```

This will set the about panel options. This will override the values defined in the app's `.plist` file on macOS. See the [Apple Docs](https://developer.apple.com/documentation/appkit/nsapplication/1428479-orderfrontstandardaboutpanelwith?preferredLanguage=occ) for more details. On Linux `GTK_ABOUT_DIALOG` is used, and values must be set in order to be shown; there are no defaults.

If you do not set credits but still wish to surface them in your app, AppKit will look for a file named "Credits.html", "Credits.rtf", and "Credits.rtfd", in that order, in the bundle returned by the NSBundle class method main. The first file found is used, and if none is found, the info area is left blank. See Apple documentation for more information.

### [MacOS] Customize the AppName on Menu Bar

The first menu item (on OS X, just next to the Apple logo) will not show as what you set. MacOS set it to CFBundleDisplayName in plist.info automatically, although you have tried to set it to a "NEW" value.

```obj-c
NSMenu* main_menu = [[NSMenu alloc] initWithTitle:@""];
[NSApp setMainMenu:main_menu];
[main_menu addItemWithTitle:@"NEW"
                            action:nil
                            keyEquivalent:@""];
```

This App will never show as expected. At a certain time (don't knows when, maybe somewhere before applicationDidFinishLaunching and after viewDidLoad which is tested in a simple demo Cocoa App), the sytem override the item to default name "test".

However, we find a way to rename it to the value we expected. By doing the following steps at a **good** time, we can successfully set it to expected value.

```obj-c
NSMenu* root_menu = [NSApp mainMenu];
NSMenu* app_menu = [[root_menu itemAtIndex:0] submenu];
{
    // system will not update if it is not changed (the value inside).
    [app_menu setTitle:@"reset"];
}
[app_menu setTitle:@"NEW"];
```

Do notice that it might not work if you do it at an very early time, so process again after the app override it to CFBundleDisplayName.

Do also notice what we had done in brackets: we set it to a "reset" status first. We notice that the code above might not work without this step in a certain situation: we already set it to "NEW"(expected name) at an early time, and system set it to "test"(CFBundleDisplayName) after my first setting (we do not know which will happen first). We detected this change (or not) at runtime and want to do it again to correct it, which fails to work if do so without a "reset". We suspect that the system stored the value we previously set. While we set it to "NEW" a second time, it compare the value with the title stored: no changes detected. So the system just skip the change. So, we add the steps in brackets to ensure system rerender the menu bar.

### [MasOS] Disable a Menu Item on Menu Bar

[validateMenuItem](https://developer.apple.com/documentation/appkit/nsmenuitemvalidation/3005191-validatemenuitem?language=objc) is implemented to override the default action of enabling or disabling a specific menu item.

The object implementing this method must be the target of menuItem. You can determine which menu item menuItem is by querying it for its tag or action.

```obj-c
@interface handler : NSObject {}
- (BOOL)validateMenuItem:(NSMenuItem*)item;
@end
@implementation handler
- (BOOL)validateMenuItem:(NSMenuItem*)item {
  SEL theAction = [item action];
  if (ItemShouldDisable(item)) {
    // SEL [item action] or tag
    return NO;
  }
  return YES;
}
@end

[menu_item setTarget:new handler()]
```