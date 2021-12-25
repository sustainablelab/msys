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

# Set up shortcuts to launch shells from PowerShell

There are two shells:

- `msys`
    - use this for package management and POSIX stuff
- `mingw`
    - use this for Windows builds (like what we's gonna do with IMGUI)
    - stuff runs faster from this shell, e.g., Python

Instead of running from Desktop shortcuts, edit the PowerShell
Profile and create aliases to launch the two shells from
PowerShell:

```
Set-Alias -Name msys -Value "C:\msys64\msys2.exe"
Set-Alias -Name mingw -Value "C:\msys64\mingw64.exe"
```

This works but it may cause problems because it bypasses the
`msys2_shell.cmd` that triggers when clicking the Desktop
shortcuts, so it's not an identical way to run the shells because
some initialization steps will not run.

To avoid confusion in the future, the right way is to run the
shell and pass the desired shell type as a flag. This requires a
slightly fancier version of the `Set-Alias` command, passing a
function name to `-Value` instead of a string:

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

# Install more packages

Open an `msys` shell.

Install these packages:

```msys-pacman-S
pacman -S make
pacman -S git
pacman -S gcc
pacman -S man
```

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

