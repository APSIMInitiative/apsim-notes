# Gtk/GtkSharp

The apsim user interface uses the Gtk+ UI toolkit. Gtk is written in C, so apsim uses a C# wrapper called [GtkSharp](https://github.com/GtkSharp/GtkSharp), which is a .net standard library which supports gtk+3. Note that [gtk-sharp](https://github.com/mono/gtk-sharp) supports only gtk2 and .net framework.

## Dependencies

The GtkSharp nuget package does not include the native libraries. Instead, these are automatically downloaded and "installed" on developer machines when building any project which references the GtkSharp nuget package. See the[installers' documentation](installers.md) for details on dependency management for the release builds.

## Debugging

When the apsim gui crashes, you will rarely get a useful stack trace because the crashes are often caused by segmentation faults in the (native) gtk code, due to our abuse of the API. Without a stack trace it's difficult to guess what's gone wrong, although the presence of any `Gtk-CRITICAL` messages in stdout from apsim are often a good clue. If this is not enough, then a native stack trace may be helpful. The best way to obtain one of these (that I've found) is to run apsim through a native debugger such as GDB. This is simple for me on Linux, although I'm not sure how well GDB runs on Windows. I leave this as an exercise to the reader. Once gdb is installed, you can start a new debugging session:

```bash
gdb /path/to/ApsimNG
```

Or attach to an existing ApsimNG process. First, find apsim's PID, then pass that to GDB:

```bash
gdb -p $PID
```

Once gdb is running, you'll need to run the attached process. Just type `run` (or `r` for short) at the GDB prompt. GDB will catch any segfaults by default, but it may also catch some other unwanted signals (such as breakpoints if you're running apsim through an IDE). To ignore these, you can run something like this at the prompt while GDB is paused at one of these events (substitute the signals you want to ignore):

```bash
handle SIG34 SIG35 nostop
```

The release builds of GTK have a lot of the debugging symbols optimized out, which can result in some very sparse stack traces; stack frames with unknown names make it difficult to know what's going on. The workaround for this is to manually compile gtk with debugging symbols included and ensure that apsim loads the newly-compiled libs. See [here](https://docs.gtk.org/gtk3/building.html) for gtk+3 compilation instructions. As for loading the correct libs, on Linux you need to add the appropriate paths to the ld search path, either by adding them to the `LD_LIBRARY_PATH` before running apsim, or, for a more persistent configuration, by creating a .conf file inside /etc/ld.so.conf.d:

```bash
echo /opt/gtk/lib >/etc/ld.so.conf.d/gtk.conf
```

Finally, it's important to bear in mind that many problems with memory management can have strange and unpredictable results, depending on what's in memory at the time. The stack trace at time of crash is not always representative of the exact cause of the problem, but it's often a good clue.

Happy debugging!
