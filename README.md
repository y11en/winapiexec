# winapiexec

winapiexec is a small tool that allows to run WinAPI functions through command
line parameters.

https://ramensoftware.com/winapiexec


winapiexec is a small tool that allows to run WinAPI functions through command line parameters.

Syntax
```
The syntax is:
winapiexec.exe library.dll@FunctionName 123 unicode_text "a space"

If you don’t specify a library or use “k”, kernel32.dll is used.
If you specify “u” as a library, user32.dll is used.

Numbers are detected automatically. You can use hex numbers (like 0xFE) and use the minus sign (e.g. -5).
Strings are Unicode by default.

You can use special prefixes to specify parameter types:
$s:ansi – an ANSI string.
$u:unicode – a Unicode string (it’s Unicode by default, but you can use it to force numbers as strings).
$b:1024 – a zero-bytes buffer with the size you specify, in bytes.
$$:1 – a reference to another parameter, you can also use $$:0 for the program’s name (argv[0]).
$a:0,1,two,3 – an array of parameters, divided by commas. You can use all the prefixes here.
$a[1,2,$a[3,4],5] – an alternative syntax for an array of parameters. Allows to have nested arrays.
$$:3@2 – a reference to an item in an array of parameters, can have more than one indirection.
```

While referencing another parameter, note that they are processed by the order of execution, which means there’s no point to reference a parameter at the right side relative to the referencing one.
Also note that when a function returns, its first parameter (like library.dll@FunctionName) is replaced with the return value.

You can execute multiple WinAPI functions, one after the other, using a comma:
winapiexec.exe library.dll@FunctionName1 123 , library.dll@FunctionName2 456
You can also have nested functions, using parentheses:
winapiexec.exe library.dll@FunctionName1 ( library.dll@FunctionName2 456 )
In this case the return value of the internal function is passed as a parameter to the external function.

Download
zip winapiexec.zip (6.02 kB)

Source code
https://github.com/m417z/winapiexec

Examples
Here are some examples of what you can do:
```
Display temp path:
winapiexec.exe GetTempPathW 260 $b:520 , u@MessageBoxW 0 $$:3 $$:0 0x40

Greetings:
winapiexec.exe advapi32.dll@GetUserNameW $b:65534 $a:32767 , u@wsprintfW $b:2050 "Hello %s from %s" $$:2 $$:0 , u@MessageBoxW 0 $$:6 ... 0

Hide the taskbar for half a second, then show it:
winapiexec.exe u@ShowWindow ( u@FindWindowW Shell_TrayWnd 0 ) 0 , Sleep 500 , u@ShowWindow $$:3 5

Run Notepad for a second, then terminate it:
winapiexec.exe CreateProcessW 0 notepad.exe 0 0 0 0x20 0 0 $a:0x44,,,,,,,,,,,,,,,, $b:16 , Sleep 1000 , TerminateProcess $$:11@0 0

Show a message box and then create a new instance of the process:
winapiexec.exe u@MessageBoxW 0 Hello! :) 0 , CreateProcessW $$:0 ( GetCommandLineW ) 0 0 0 0x20 0 0 $a:0x44,,,,,,,,,,,,,,,, $b:16

Eject your disc drive 🙂
winapiexec.exe winmm.dll@mciSendStringW "open cdaudio" 0 0 0 , winmm.dll@mciSendStringW "set cdaudio door open" 0 0 0 , winmm.dll@mciSendStringW "close cdaudio" 0 0 0

And several more practical examples…

Copy some text into the clipboard:
winapiexec.exe lstrcpyW ( GlobalLock ( GlobalAlloc 0x0042 8192 ) ) "Sample text" , GlobalUnlock $$:5 , u@OpenClipboard 0 , u@SetClipboardData 13 $$:5 , u@CloseClipboard

Turn off and on monitor:
winapiexec.exe u@SendMessageW 0xFFFF 0x112 0xF170 2
winapiexec.exe u@SendMessageW 0xFFFF 0x112 0xF170 -1

Clear the icon cache:
winapiexec.exe shell32.dll@SHChangeNotify 0x08000000 0 0 0

Display the Start menu:
winapiexec.exe u@SendMessageW ( u@FindWindowW Shell_TrayWnd 0 ) 0x111 305 0
Run task manager:
winapiexec.exe u@SendMessageW ( u@FindWindowW Shell_TrayWnd 0 ) 0x111 420 0
```
More tricks like the last two can be found here.
