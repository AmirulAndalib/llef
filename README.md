<p align="center">
  <img src="assets/llef-dragon-small.png" alt="llef logo"/>
</p>

# LLEF

LLEF (pronounced ɬɛf - "hlyeff") is an LLDB plugin to make it more usable for low-level RE and VR. Similar to [GEF](https://github.com/hugsy/gef), but for LLDB.

It uses LLDB's Python API to add extra status output and a few new commands, so that security researchers can more easily use LLDB to analyse software as it's running.

![llef demo](https://foundryzero.co.uk/assets/img/llef-small.gif)

## 💻 Supported Architectures
* x86_64
* arm
* aarch64 / arm64
* i386

## 📓 Requirements
* LLDB 15+ (https://apt.llvm.org/) _On macOS this is bundled with Xcode 14.3+_

## ⚙ Installation
The instructions below will install LLEF so that it is used by LLDB by default.

1. Clone the repository.
2. `cd <repo>`
3. Run `./install.sh`
4. Select automatic (overwrites `~/.lldbinit`) or manual installation.

_LLDB uses AT&T disassembly syntax for x86 binaries by default. The installer provides an option to override this._

## ▶ Usage

### Launch LLDB

```bash
lldb-15 <optional binary to debug>
```

### Use commands:

#### llefsettings
Various commands for setting, saving, loading and listing LLEF specific commands:
```
(lldb) llefsettings --help
list                list all settings
save                Save settings to config file
reload              Reload settings from config file (retain session values)
reset               Reload settings from config file (purge session values)
set                 Set LLEF settings
```

Settings are stored in a file `.llef` located in your home directory formatted as following:
```
[LLEF]
<llefsettings> = <value>
```

##### Available Settings

| Setting           | Type    | Description                                        |
|-------------------|---------|----------------------------------------------------|
| color_output      | Boolean | Enable/disable color terminal output               |
| register_coloring | Boolean | Enable/disable register coloring                   |
| show_legend       | Boolean | Enable/disable legend output                       |
| show_registers    | Boolean | Enable/disable registers output                    |
| show_stack        | Boolean | Enable/disable stack output                        |
| show_code         | Boolean | Enable/disable code output                         |
| show_threads      | Boolean | Enable/disable threads output                      |
| show_trace        | Boolean | Enable/disable trace output                        |
| force_arch        | String  | Force register display architecture (experimental) |
| rebase_addresses  | Boolean | Enable/disable address rebase output               |
| rebase_offset     | Int     | Set the rebase offset (default 0x100000)           |

#### Context

Refresh the LLEF GUI with:
```
(lldb) context
```

#### Pattern Create
```
(lldb) pattern create 10
[+] Generating a pattern of 10 bytes (n=4)
aaaabaaaca
[+] Pattern saved in variable: $8
(lldb) pattern create 100 -n 2
[+] Generating a pattern of 100 bytes (n=2)
aabacadaea
[+] Pattern saved in variable: $9
```

#### Pattern Search

```
(lldb) pattern search $rdx
[+] Found in $10 at index 45 (big endian)
(lldb) pattern search $8
[+] Found in $10 at index 0 (little endian)
(lldb) pattern search aaaabaaac
[+] Found in $8 at index 0 (little endian)
(lldb) pattern search 0x61616161626161616361
[+] Found in $8 at index 0 (little endian)
```


### Breakpoint hook
This is automatic and prints all the currently implemented information at a break point.

#### Address Rebasing
Configurable with the `rebase_addresses` setting the address rebasing feature performs a lookup for each code address presented in the output to display the associated binary and relative address. This relative address is offset by the value defined in setting `rebase_offset` which defaults to the Ghidra base address of `0x100000`. The result is an address output that can be easily copied and pasted into an IDE "Go To Address" feature without having to do the maths to convert from the runtime address.

Rebased addresses are shown in brackets after the runtime address:
![rebase address feature](assets/rebase-feature.png)

## 👷‍♂️ Troubleshooting LLDB Python support
LLDB comes bundled with python modules that are required for LLEF to run. If on launching LLDB with LLEF you encounter `ModuleNotFoundError` messages it is likely you will need to manually add the LLDB python modules on your python path.

To do this run the following to establish your site-packages location:

```bash
python3 -m site --user-site
```

Then locate the LLDB python modules location. This is typically at a location such as `/usr/lib/llvm-15/lib/python3.10/dist-packages` but depends on your python version.

Finally, modify and execute the following to add the above LLDB module path into a new file `lldb.pth` in the site-packages location discovered above.

```bash
echo "/usr/lib/llvm-15/lib/python3.10/dist-packages" > ~/.local/lib/python3.10/site-packages/lldb.pth
```

## Performance Optimisations

Rendering LLEF output at each breakpoint has been observed to be slow on some platforms. The root cause of this has been traced to the underlying `GetMemoryRegions` LLDB API call. Fortunately, this is only used to identify to whether register values point to code, stack or heap addresses.

To disable register coloring, and potentially significantly improve LLEF performance, disable the `register_coloring` feature using the following `llefsettings` command.

```
 llefsettings set register_coloring False
 ```


## 👏 Thanks
We’re obviously standing on the shoulders of giants here - we’d like to credit [hugsy](https://twitter.com/_hugsy_) for [GEF](https://github.com/hugsy/gef) in particular, from which this tool draws *heavy* inspiration! Please consider this imitation as flattery 🙂

If you'd like to read a bit more about LLEF you could visit our [launch blog post](https://foundryzero.co.uk/2023/07/13/llef.html).
