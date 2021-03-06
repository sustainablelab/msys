From the MSYS2 website:

> MSYS2 is a collection of tools and libraries providing you with
> an easy-to-use environment for building, installing and running
> native Windows software.

- [Install MSYS2](README.md#install-msys2)
- [Set up shortcuts to launch shells from PowerShell](README.md#set-up-shortcuts-to-launch-shells-from-powershell)
- [Install more packages](README.md#install-more-packages)
- [Test gcc](README.md#test-gcc)

# Install MSYS2

- download from https://www.msys2.org/
- take one (or both) of these precautions:
    - check the hash:
        - calculate the SHA256 on the downloaded file
        - PowerShell has the built-in command `Get-FileHash`

            ```
            Get-FileHash .\msys2-x86_64-20210604.exe
            2E9BD59980AA0AA9248E5F0AD0EF26B0AC10ADAE7C6D31509762069BB388E600
            ```

        - check the hash matches the SHA256 checksum on the website:

            ```
            $(Get-FileHash .\msys2-x86_64-20210604.exe).Hash -match "2e9bd59980aa0aa9248e5f0ad0ef26b0ac10adae7c6d31509762069bb388e600"
            ```

    - check the signature:
        - if you already have Cygwin installed on this computer
          use that to run `gpg`, if not download to a Linux
          computer and use `gpg` there, and if not that, the hash
          is good enough
        - it is not worth setting up `gpg` to do this from
          PowerShell
        - if you do have `gpg`:
            - use `gpg` to import the public key listed on the
              msys2.org website:

              ```
              gpg --recv-key 0xf7a49b0ec
              ```

            - download the GPG signature (should be a link just
              before the public key)
            - check that the downloaded file was signed with that
              key:

            ```
            gpg --keyid-format=long --with-fingerprint --verify msys2-x86_64-20210604.exe.sig msys2-x86_64-20210604.exe
            ```

            - look for "Good signature from"
            - don't worry about the warning: "WARNING: This key
              is not certified with a trusted signature!"

        - once MSYS2 is installed, you can run `gpg` from the
          `msys` shell to check signatures in the future

- run installer
- check the installation is OK:
    - MSYS will launch a shell
    - `ls -a` here: MSYS should have installed a default
      `.bashrc`
    - if there is no `.bashrc` file, something is probably wrong
      with the `$HOME` environment variable
- I ran into this on a work laptop:
    - Allegro Cadence programs create an environment variable
      named `$HOME` (for no good reason)
        - checking `$HOME` from PowerShell does not show this
          modified value
        - the only way to see it is to open the Windows GUI that
          shows the Environment Variables
        - there should not be a `HOME` variable shown there
            - if there is a `HOME`, it was likely put there by a
              program (like one of the Allegro Cadence programs)
- the fix is simple:
    - delete the `$HOME` environment variable (close any
      offending programs first, like the Allegro Cadence
      programs)
    - run the `msys` uninstaller
    - install `msys` again

Open the MSYS2 `msys` shell and run the package manager to update
system packages:

```
pacman -Syu
```

The `-y` flag downloads all the needed system packages. `-u`
upgrades.

Run the `-u` flag again to make sure everything is installed:

```
pacman -Su
```

Read more about package management here:

https://www.msys2.org/docs/package-management/

List all the top-level pacman flags (capital letters):

```
pacman -h
```

List the lower-level (lowercase letter) flags for any top-level
(capital letter) pacman flag:

```
pacman -S --help
pacman -Q --help
pacman -F --help
```

For example, `-S` and `-F` both have a `-y` flag, but `-Q` does
not have a `-y` flag.

The help for `pacman -S` or `pacman -F` says:

```
-y, --refresh        download fresh package databases from the server
```

So `pacman -Fy` and `pacman -Sy` both update the local copy of the package
databases:

```
$ pacman -Fy
:: Synchronizing package databases...
 mingw32                 4.9 MiB  2.84 MiB/s 00:02 [#####################] 100%
 mingw64                 4.9 MiB  3.54 MiB/s 00:01 [#####################] 100%
 ucrt64                  5.0 MiB  3.36 MiB/s 00:01 [#####################] 100%
 clang64                 4.6 MiB  2.61 MiB/s 00:02 [#####################] 100%
 msys                 1061.7 KiB   427 KiB/s 00:02 [#####################] 100%
error: failed retrieving file 'msys.files' from downloads.sourceforge.net : The requested URL returned error: 404
```

```
$ pacman -Sy
:: Synchronizing package databases...
 mingw32 is up to date
 mingw64 is up to date
 ucrt64 is up to date
 clang64 is up to date
 msys is up to date
error: failed retrieving file 'msys.db' from downloads.sourceforge.net : The requested URL returned error: 404
```

Some examples. I use `emacs` as a package name search term in
these examples.


Search for a package in the repositories:

```
$ pacman -Ss emacs
mingw32/mingw-w64-i686-emacs 28.1-1
    The extensible, customizable, self-documenting, real-time display editor (mingw-w64)
mingw32/mingw-w64-i686-emacs-pdf-tools-server 0.91-1
    Emacs support library for PDF files
mingw32/mingw-w64-i686-liberime 0.0.6-1
    An emacs dynamic module provide librime bindings for emacs (mingw-w64)
mingw64/mingw-w64-x86_64-emacs 28.1-1 [installed]
    The extensible, customizable, self-documenting, real-time display editor (mingw-w64)
mingw64/mingw-w64-x86_64-emacs-pdf-tools-server 0.91-1
    Emacs support library for PDF files
mingw64/mingw-w64-x86_64-liberime 0.0.6-1
    An emacs dynamic module provide librime bindings for emacs (mingw-w64)
ucrt64/mingw-w64-ucrt-x86_64-liberime 0.0.6-1
    An emacs dynamic module provide librime bindings for emacs (mingw-w64)
clang64/mingw-w64-clang-x86_64-liberime 0.0.6-1
    An emacs dynamic module provide librime bindings for emacs (mingw-w64)
msys/cmake-emacs 3.22.1-2
    A cross-platform open-source make system (Emacs mode)
msys/emacs 27.2-1 (editors)
    The extensible, customizable, self-documenting, real-time display editor (msys2)
msys/ninja-emacs 1.10.2-1
    Ninja is a small build system with a focus on speed (Emacs mode)
```

Narrow that search:

```
$ pacman -Ss 64-emacs
mingw64/mingw-w64-x86_64-emacs 28.1-1 [installed]
    The extensible, customizable, self-documenting, real-time display editor (mingw-w64)
mingw64/mingw-w64-x86_64-emacs-pdf-tools-server 0.91-1
    Emacs support library for PDF files
```

*Note the already installed package shows as `[installed]`.*

Search for a package in the installed packages:

```
$ pacman -Qs emacs
local/mingw-w64-x86_64-emacs 28.1-1
    The extensible, customizable, self-documenting, real-time display editor (mingw-w64)
```

Use `pactree` to list dependencies (have to use full package name
for this one):

```
$ pactree mingw-w64-x86_64-emacs
mingw-w64-x86_64-emacs
??????mingw-w64-x86_64-universal-ctags-git
??? ??????mingw-w64-x86_64-gcc-libs
...
??? ??????mingw-w64-x86_64-zlib
??? ??????mingw-w64-x86_64-libwinpthread-git
??????mingw-w64-x86_64-libwinpthread-git provides mingw-w64-x86_64-libwinpthread
```


Find out which package a file belongs to (note this particular
command must be run from `mingw`, not `msys` because `msys`
doesn't know what `emacs.exe` is):

```
$ pacman -Qo emacs.exe
/mingw64/bin/emacs.exe is owned by mingw-w64-x86_64-emacs 28.1-1
```

To run that same command from `msys` I have to give the full
path to the file in question:

```
$ pacman -Qo /mingw64/bin/emacs.exe
/mingw64/bin/emacs.exe is owned by mingw-w64-x86_64-emacs 28.1-1
```


# Set up shortcuts to launch shells from PowerShell

MSYS2 provides three *subsystems*: `msys2`, `mingw32`, and
`mingw64`.

- I use subsystems `msys2` and `mingw64`.
- I make PowerShell aliases named `msys` and `mingw`
    - the aliases launch the **shell** associated with the
      subsystem
    - `msys` launches the shell for `msys2`
    - `mingw` launches the shell for `mingw64`

Use `msys` to manage packages and `mingw` for programming:

- `msys`
    - I only use this environment for maintenance
    - use this environment for package management and POSIX stuff
- `mingw`
    - most of the time I am working in this environment
    - use this environment for building Windows executables
        - e.g., building an application that uses IMGUI
    - stuff runs faster from this shell, e.g., Python

Set up the aliases this way:

```
function RunMsys {
    C:\msys64\msys2_shell.cmd -msys
}
Set-Alias -Name msys -Value RunMsys

function RunMingw {
    C:\msys64\msys2_shell.cmd -mingw
}
Set-Alias -Name mingw -Value RunMingw
```

Don't set up the aliases like this:

```PowerShell
Set-Alias -Name msys -Value "C:\msys64\msys2.exe"
Set-Alias -Name mingw -Value "C:\msys64\mingw64.exe"
```

This works but it may cause problems because it bypasses the
`msys2_shell.cmd` that triggers when clicking the Desktop
shortcuts, so it's not an identical way to run the shells because
some initialization steps will not run.

**The right way is to run the shell and pass the desired shell type
as a flag.**

This is why I create PowerShell functions and pass the function
name as the `-Value` to the `Set-Alias` command (instead of
passing the string path to the `.exe`).

# Install more packages

Open an `msys` shell.

Install these packages:

```msys-pacman-S
pacman -S make
pacman -S git
pacman -S subversion
pacman -S python
pacman -S man
pacman -S pkgconf
pacman -S gcc
```

## Install Emacs

To install Emacs:

```bash
pacman -S mingw-w64-x86_64-emacs
```

To launch Emacs:

```bash
emacs
```

### Emacs in terminal mode on Windows

**Don't run Emacs in terminal mode.**

*But here are some instructions to try terminal mode once.*

By default, Emacs runs in GUI mode. Use flag `-nw` to run in
terminal mode.

```bash
winpty emacs -Q -nw
```

What is the `winpty` and the `-Q`?

#### emacs -Q

The `-Q` flag starts Emacs in the **vanilla mode**.
That means it is out-of-the-box Emacs with no customizations.
Disabling all customization is a good idea when trying out
terminal mode.

- **vanilla** means ignore any config in `.emacs.d/`
    - Emacs starts up faster -- no checking for packages
    - that is especially important when running in terminal
      mode, especially with **spacemacs** which checks for a
      lot of packages
    - during this package checking, if I am in terminal mode,
      the keyboard input to all other terminal applications
      is ridiculously slow, like 1-second per keystroke
- **vanilla** means the menubar is there
    - the terminal "eats" a lot of Ctrl-key combinations
    - crucially C-c is intercepted by the terminal
        - so I cannot use C-x/C-c for quit
        - the only other way I know how to quit is `File -> Quit`
    - but if I am not in vanilla emacs, the menubar is turned off
      (because no experience Emacs user uses the menubar)
    - then the only way to quit Emacs is to kill the entire
      window/process

#### winpty

Try to launch terminal mode without the `winpty`. I get this
error:

```bash
$ emacs -Q -nw
emacs: standard input is not a tty
```

From:

https://github.com/mintty/mintty/wiki/Tips#inputoutput-interaction-with-alien-programs

> As a workaround on older versions of Cygwin or Windows, you can
> use [winpty](https://github.com/rprichard/winpty) as a wrapper
> to invoke the Windows program.
>
> The same workaround handles interrupt signals, particularly
> Control+C, which does not otherwise function as expected with
> non-cygwin programs.

What is a `tty`? That stands for "teletype". No one uses
teletypes anymore. A teletype is a terminal attached to the
computer's serial port. This serial terminal has a keyboard for
sending input to the computer, and it has a printer for receiving
output from the computer.

Sometimes operating systems communicate with serial port devices
as if they are a `tty` (an external device sending text in and
expecting text back). But when people talk about a `tty` that is
running *on* the computer, they really mean a *terminal
emulator*, not an external machine communicating over the serial
port. In this usage, the terms `tty`, terminal, `pty`, and
terminal emulator are all interchangeable.

The `bash` shells in `MinGW`, `MSYS`, and `Cygwin` all run in the
`mintty` terminal emulator. Some other terminal emulators are
`PuTTY` and `winpty`.

So what? I can *launch* any executable from a terminal emulator.
At that point the OS is going to execute the executable. It
doesn't matter that I launched it from a terminal.

The issue launching Emacs in terminal mode is that this `-nw`
flag expects a terminal. In other words, I need to open a
terminal emulator to act as a terminal *to the executable*.

**On Windows, this means the terminal emulator has to present
itself as a "console".**

I don't know if `mintty` can do this. If it can, I don't know how
to do it. But `winpty`, yet another terminal emulator, does
present itself as a console. The trouble with `winpty` is that it
eats the interrupt signals (like Control+C).

## Install Vim

To install Vim:

```msys-pacman-S
pacman -S vim
```

Clone my private `vim-dotvim` repo into folder `.vim`. Then, to
fully replicate my Vim setup, I clone all the Vim plug-ins like
this:

```bash
cd .vim
git submodule init
git submodule update
```

On my desktop, the color for `highlight SpellBad` is a beautiful
light purple:

```vim
highlight SpellBad cterm=underline,bold ctermfg=4
```

But on Raspberry Pi and some laptops, this same color shows up as
an illegible dark blue. I change this to light blue by changing
the `4` to a `6`:

```vim
highlight SpellBad cterm=underline,bold ctermfg=6
```

## minttyrc

Some of the mintty settings also affect Vim appearance.

By default, there is no `.minttyrc` file. Create the file and put
this in it. This makes the cursor a non-blinking block, lets me
use `Ctrl+Shift+T` to adjust transparency, and makes the terminal
colors nice (these colors have no effect on Vim).

```minttyrc
Font=Consolas
ThemeFile=dracula
CursorType=block
BoldAsFont=yes
Locale=en_US
Charset=UTF-8
FontSmoothing=full
PgUpDnScroll=yes
Term=xterm
BellType=0
CursorBlinks=no
Language=en_US
CtrlShiftShortcuts=yes

# User-defined shortcuts (KeyFunctions=)
KeyFunctions=F1:`explorer "C:\Users\mike\Downloads"`;\
             F8:`echo -n some-text-I-need-to-write-a-lot`;
```

The shortcuts show an example of setting up keybindings. In this
example `F1` opens the Windows explorer in the Downloads folder,
and `F8` prints some frequently used text.

Don't try to run GNU utilities with a `mintty` shortcut (like a
shortcut that runs `vim` or `man`). But it seems OK to call a
`.exe` that quickly returns, like the one I show. Use the `\` to
line break the `KeyFunctions`.

## bashrc

I make few changes to the default `.bashrc`:

```bash
set -o vi
```

I make `rm` and `mv` interactive:

```bash
alias rm='rm -i'
alias mv='mv -i'
```

And I make `ls` print in color (`--color`) with classification markings
(`-F`:  `*` indicates executable and `/` indicates folder)

```bash
alias ls='ls -F --color'
```

And `lsv` for vertical list, with `-h` for human-readable file
sizes and `-l` for the vertical (long) listing:

```bash
alias lsv='ls -hlF --color'
```

And I set some aliases to call Windows executables from a Vim
terminal. For example, this lets me start VisualStudio from a Vim
terminal:

```bash
devenv="/c/Program Files (x86)/Microsoft Visual Studio/2019/Community/Common7/IDE/devenv.exe"
export devenv
alias devenv='"$devenv"'
```

And export variables so it's easy to get to certain files or
folders. For example, this makes it easy to access my PowerShell
Profile from `mintty` or `Vim`:

```bash
# from mintty: vim $profile
# from vim: :e $profile
profile="/c/Users/mike/Documents/WindowsPowerShell/Microsoft.PowerShell_profile.ps1"
export profile
```

## Install Python

Install Python:

```msys-pacman-S
pacman -S python
```

I do this even if Python is already installed on Windows. I
cannot run the Windows Python from the `msys` or `mingw` shells.
This is not simply because the `$env:PATH` gets stripped. Even if
I inherit the full Windows `$PATH`, the Windows Python install
just doesn't run, don't know why.

Further, I like to source Python scripts from Vim. For some
reason, the command to do this is slightly different in Cygwin
bash and MinGW bash.

I have to change two things in my `.vimrc`:

1. `;so` (my shortcut for sourcing a script):
    - I use `!./%` under Cygwin
    - change it to `!%` under MinGW
2. sourcing a file works because of the Python shebang
    - Vim inserts the shebang for me
    - I use `usr/bin/env python3` under Cygwin
    - change it to `usr/bin/python` under MinGW

# Package management

## Do your own updates

Maintain the MSYS2 installation yourself. There is no automatic
update checker.

Run this command:

```bash
pacman -Syu
```

Then repeat this command until you get the message there is
nothing to do:

```bash
pacman -Su
```

## Use pacman

To check which packages you explicitly installed:

```bash
pacman -Qe
```


# Test gcc

Create a dummy C file to test gcc:

```c
/* test.c */
int main(void){}
```

Generate assembly code:

```bash
gcc -S test.c
```

After the word `gcc`, the order of the rest doesn't matter, so
this is OK too:

```bash
gcc test.c -S
```

This generates the following `test.s`:

```asm
	.file	"test.c"
	.text
	.def	__main;	.scl	2;	.type	32;	.endef
	.globl	main
	.def	main;	.scl	2;	.type	32;	.endef
	.seh_proc	main
main:
.LFB0:
	pushq	%rbp
	.seh_pushreg	%rbp
	movq	%rsp, %rbp
	.seh_setframe	%rbp, 0
	subq	$32, %rsp
	.seh_stackalloc	32
	.seh_endprologue
	call	__main
	movl	$0, %eax
	addq	$32, %rsp
	popq	%rbp
	ret
	.seh_endproc
	.ident	"GCC: (Rev5, Built by MSYS2 project) 10.3.0"
```

Alternatively, this outputs to the terminal:

```bash
gcc test.c -S -o -
```

The `-o` says output to file, e.g., `-o test.s`. Instead of
giving a file name, I put `-` which means output to the terminal.

The assembly is meant for conversion by the GNU tool `as.exe`
into an object file. The instructions, e.g., `movl` and `popq`,
are machine-independent. The directives, e.g., `seh_proc`
(a Windows thing that stands for "structured exception
handling"), are machine-dependent.

