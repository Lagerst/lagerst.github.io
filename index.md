## Welcome

### Tada

	 /***
	 *                                         ,s555SB@@&
	 *                                      :9H####@@@@@Xi
	 *                                     1@@@@@@@@@@@@@@8
	 *                                   ,8@@@@@@@@@B@@@@@@8
	 *                                  :B@@@@X3hi8Bs;B@@@@@Ah,
	 *             ,8i                  r@@@B:     1S ,M@@@@@@#8;
	 *            1AB35.i:               X@@8 .   SGhr ,A@@@@@@@@S
	 *            1@h31MX8                18Hhh3i .i3r ,A@@@@@@@@@5
	 *            ;@&i,58r5                 rGSS:     :B@@@@@@@@@@A
	 *             1#i  . 9i                 hX.  .: .5@@@@@@@@@@@1
	 *              sG1,  ,G53s.              9#Xi;hS5 3B@@@@@@@B1
	 *               .h8h.,A@@@MXSs,           #@H1:    3ssSSX@1
	 *               s ,@@@@@@@@@@@@Xhi,       r#@@X1s9M8    .GA981
	 *               ,. rS8H#@@@@@@@@@@#HG51;.  .h31i;9@r    .8@@@@BS;i;
	 *                .19AXXXAB@@@@@@@@@@@@@@#MHXG893hrX#XGGXM@@@@@@@@@@MS
	 *                s@@MM@@@hsX#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&,
	 *              :GB@#3G@@Brs ,1GM@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@B,
	 *            .hM@@@#@@#MX 51  r;iSGAM@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@8
	 *          :3B@@@@@@@@@@@&9@h :Gs   .;sSXH@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@:
	 *      s&HA#@@@@@@@@@@@@@@M89A;.8S.       ,r3@@@@@@@@@@@@@@@@@@@@@@@@@@@r
	 *   ,13B@@@@@@@@@@@@@@@@@@@5 5B3 ;.         ;@@@@@@@@@@@@@@@@@@@@@@@@@@@i
	 *  5#@@#&@@@@@@@@@@@@@@@@@@9  .39:          ;@@@@@@@@@@@@@@@@@@@@@@@@@@@;
	 *  9@@@X:MM@@@@@@@@@@@@@@@#;    ;31.         H@@@@@@@@@@@@@@@@@@@@@@@@@@:
	 *   SH#@B9.rM@@@@@@@@@@@@@B       :.         3@@@@@@@@@@@@@@@@@@@@@@@@@@5
	 *     ,:.   9@@@@@@@@@@@#HB5                 .M@@@@@@@@@@@@@@@@@@@@@@@@@B
	 *           ,ssirhSM@&1;i19911i,.             s@@@@@@@@@@@@@@@@@@@@@@@@@@S
	 *              ,,,rHAri1h1rh&@#353Sh:          8@@@@@@@@@@@@@@@@@@@@@@@@@#:
	 *            .A3hH@#5S553&@@#h   i:i9S          #@@@@@@@@@@@@@@@@@@@@@@@@@A.
	 *
	 *
	 */

### How to set Menu Bar App Name on MacOS

The first menu item (on OS X, just next to the Apple logo) will not show as what you set. MacOS set it to CFBundleDisplayName in plist.info automatically, although you tried to set it to "NEW".

```obj-c
    NSMenu* main_menu = [[NSMenu alloc] initWithTitle:@""];
    [NSApp setMainMenu:main_menu];
    [main_menu addItemWithTitle:@"NEW"
                                action:nil
                                keyEquivalent:@""];
```

This App will never show as expected. At a certain time (nobody knows when, maybe somewhere before applicationDidFinishLaunching and after viewDidLoad which is tested in a simple demo Cocoa App), the sytem override the item to default name "test". However, we find a way to rename it to the value we expected.

```obj-c
    NSMenu* root_menu = [NSApp mainMenu];
    NSMenu* app_menu = [[root_menu itemAtIndex:0] submenu];
    {
        // system will not update if it is not changed (the value inside).
        [app_menu setTitle:@"reset"];
    }
    [app_menu setTitle:@"NEW"];
```

By doing so at a **good** time, we can successfully set it to expected value. But notice what we had done in brackets: we set it to a "reset" status first. We notice that this might not work without this step in a certain situation: we already set it to "NEW"(expected name) at an early time, but system set it to "test"(CFBundleDisplayName) after my first setting. We detected (or not) this change at runtime and want to do it again to correct it, which fails to work. 
We suspect that the system stored the value we previously set, while we set it to "NEW" a second time, it compare the value with the title stored: no changes detected. So the system just skip the change. So, we add the steps in brackets to ensure system rerender the menu bar.

### How to launch a Privileged Task on Mac

Use [STPrivilegedTask](https://github.com/sveinbjornt/STPrivilegedTask) which is deprecated but can still be used, or [SMJobBless](https://developer.apple.com/library/archive/samplecode/EvenBetterAuthorizationSample/Introduction/Intro.html) which is complicated but less likely to be deprecated in the future.

Special Notes:
On posix system files created by privieged task via base::File::FLAG_CREATE in chromium/src/base/file.h will be locked for only root user to aceess, since file is created with default mode
```c
    int mode = S_IRUSR | S_IWUSR;
```
which only allow the current user to read and write. Unfortunately, the current user is root, and when normal user launch the app it cannot access the file and may never able to read/write the file, which is fatal to some settings stored in local disk.
So avoid such operation in privileged tasks, some files may be overwriten by deleting & recreating to overwritten, so they will also have the problem.
