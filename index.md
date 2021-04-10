## Welcome

### Tada
```
                                      ,s555SB@@&
                                    :9H####@@@@@Xi
                                   1@@@@@@@@@@@@@@8
                                 ,8@@@@@@@@@B@@@@@@8
                                :B@@@@X3hi8Bs;B@@@@@Ah,
           ,8i                  r@@@B:     1S ,M@@@@@@#8;
          1AB35.i:               X@@8 .   SGhr ,A@@@@@@@@S
          1@h31MX8                18Hhh3i .i3r ,A@@@@@@@@@5
          ;@&i,58r5                 rGSS:     :B@@@@@@@@@@A
           1#i  . 9i                 hX.  .: .5@@@@@@@@@@@1
            sG1,  ,G53s.              9#Xi;hS5 3B@@@@@@@B1
             .h8h.,A@@@MXSs,           #@H1:    3ssSSX@1
             s ,@@@@@@@@@@@@Xhi,       r#@@X1s9M8    .GA981
             ,. rS8H#@@@@@@@@@@#HG51;.  .h31i;9@r    .8@@@@BS;i;
              .19AXXXAB@@@@@@@@@@@@@@#MHXG893hrX#XGGXM@@@@@@@@@@MS
              s@@MM@@@hsX#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&,
            :GB@#3G@@Brs ,1GM@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@B,
          .hM@@@#@@#MX 51  r;iSGAM@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@8
        :3B@@@@@@@@@@@&9@h :Gs   .;sSXH@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@:
    s&HA#@@@@@@@@@@@@@@M89A;.8S.       ,r3@@@@@@@@@@@@@@@@@@@@@@@@@@@r
 ,13B@@@@@@@@@@@@@@@@@@@5 5B3 ;.         ;@@@@@@@@@@@@@@@@@@@@@@@@@@@i
5#@@#&@@@@@@@@@@@@@@@@@@9  .39:          ;@@@@@@@@@@@@@@@@@@@@@@@@@@@;
9@@@X:MM@@@@@@@@@@@@@@@#;    ;31.         H@@@@@@@@@@@@@@@@@@@@@@@@@@:
 SH#@B9.rM@@@@@@@@@@@@@B       :.         3@@@@@@@@@@@@@@@@@@@@@@@@@@5
   ,:.   9@@@@@@@@@@@#HB5                 .M@@@@@@@@@@@@@@@@@@@@@@@@@B
         ,ssirhSM@&1;i19911i,.             s@@@@@@@@@@@@@@@@@@@@@@@@@@S
            ,,,rHAri1h1rh&@#353Sh:          8@@@@@@@@@@@@@@@@@@@@@@@@@#:
          .A3hH@#5S553&@@#h   i:i9S          #@@@@@@@@@@@@@@@@@@@@@@@@@A.
```

### [Windows] AppUserModelID

The application id (System.AppUserModel.ID property) of a Windows 7 shortcut.

In Windows 7 (or above), taskbar items are grouped by a string known as the application id or AppId. This can be set in the shortcut that launches a program, or by the application itself. See [AppUserModelIDs](http://msdn.microsoft.com/en-us/library/dd378459%28VS.85%29.aspx) for more information. There's a [tool](https://code.google.com/archive/p/win7appid/) to modify the AppUserModelID, from where we can learn to modify it in our own code.

```c++
  EXTERN_C const PROPERTYKEY DECLSPEC_SELECTANY PKEY_AppUserModel_ID = {
      {0x9F4C2855,
       0x9F79,
       0x4B39,
       {
           0xA8,
           0xD0,
           0xE1,
           0xD4,
           0x2D,
           0xE1,
           0xD5,
           0xF3,
       }},
      5};

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

### [MacOS] Launch a Privileged Task

Use [STPrivilegedTask](https://github.com/sveinbjornt/STPrivilegedTask) which is deprecated but can still be used, or [SMJobBless](https://developer.apple.com/library/archive/samplecode/EvenBetterAuthorizationSample/Introduction/Intro.html) which is complicated but less likely to be deprecated in the future.

Special Notes:
On posix system files created by privieged task via base::File::FLAG_CREATE in chromium/src/base/file.h will be locked for only root user to aceess, since file is created with default mode
```c
int mode = S_IRUSR | S_IWUSR;
```
which only allow the current user to read and write. Unfortunately, the current user is root, and when normal user launch the app it cannot access the file and may never able to read/write the file, which is fatal to some settings stored in local disk.
So avoid such operation in privileged tasks, some files may be overwriten by deleting & recreating to overwritten, so they will also have the problem.

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