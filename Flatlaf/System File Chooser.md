
# System File Choosersince v3.7

Class `SystemFileChooser` (in package `com.formdev.flatlaf.util`) allows using **operating system file dialogs** in Java Swing applications.
## Description

There are some limitations and incompatibilities to [`JFileChooser`](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JFileChooser.html) because operating system file dialogs do not offer all features that `JFileChooser` provides. On the other hand, operating system file dialogs offer features out of the box that `JFileChooser` do not offer (e.g. ask for overwrite on save). So this class offers only features that are available on all platforms.

The API is (mostly) compatible with [`JFileChooser`](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JFileChooser.html). To use `SystemFileChooser` in existing code, do a string replace from `JFileChooser` to `SystemFileChooser`. If there are no compile errors, then there is a good chance that it works without further changes. If there are compile errors, then you're using a feature that `SystemFileChooser` does not support.

Supported platforms are **Windows 10+**, **macOS 10.14+** and **Linux with GTK 3**. `JFileChooser` is used on unsupported platforms or if GTK 3 is not installed.

`SystemFileChooser` requires FlatLaf native libraries (usually contained in `flatlaf.jar`). If not available or disabled (via system property [`flatlaf.useNativeLibrary`](https://www.formdev.com/flatlaf/system-properties#flatlaf.useNativeLibrary) or [`flatlaf.useSystemFileChooser`](https://www.formdev.com/flatlaf/system-properties#flatlaf.useSystemFileChooser)), then `JFileChooser` is used.

To improve user experience, it is recommended to use a [state storage](https://www.formdev.com/flatlaf/system-file-chooser/#state_storage), so that file dialogs open at previously visited folder.

## Usage

### Open Single File

```java
SystemFileChooser fc = new SystemFileChooser();
if( fc.showOpenDialog( this ) == SystemFileChooser.APPROVE_OPTION ) {
    File file = fc.getSelectedFile();
    System.out.println( file );
}
```


### Open Multiple Files

```java
SystemFileChooser fc = new SystemFileChooser();
fc.setMultiSelectionEnabled( true );
if( fc.showOpenDialog( this ) == SystemFileChooser.APPROVE_OPTION ) {
    File[] files = chooser.getSelectedFiles();
    System.out.println( Arrays.toString( files ).replace( ",", "\n" ) );
}
```

### Save File

```java
SystemFileChooser fc = new SystemFileChooser();
if( fc.showSaveDialog( this ) == SystemFileChooser.APPROVE_OPTION ) {
    File file = fc.getSelectedFile();
    System.out.println( file );
}
```

### Select Folder

```java
SystemFileChooser fc = new SystemFileChooser();
fc.setFileSelectionMode( SystemFileChooser.DIRECTORIES_ONLY );
if( fc.showOpenDialog( this ) == SystemFileChooser.APPROVE_OPTION ) {
    File directory = fc.getSelectedFile();
    System.out.println( directory );
}

```
### File Filters

```java
fc.addChoosableFileFilter(
    new SystemFileChooser.FileNameExtensionFilter( "Text Files", "txt", "md" ) );
fc.addChoosableFileFilter(
    new SystemFileChooser.FileNameExtensionFilter( "PDF Files", "pdf" ) );
fc.addChoosableFileFilter(
    new SystemFileChooser.FileNameExtensionFilter( "Archives", "zip", "tar", "jar", "7z" ) );
```

### Approve Callback

```java
fc.setApproveCallback( (selectedFiles, context) -> {
    // do something
    return SystemFileChooser.APPROVE_OPTION; // or SystemFileChooser.CANCEL_OPTION
} );
```

or

```java
fc.setApproveCallback( this::approveCallback );

private boolean approveCallback( File[] selectedFiles, ApproveContext context ) {
    // do something
    return SystemFileChooser.APPROVE_OPTION; // or SystemFileChooser.CANCEL_OPTION
}
```

Sets a callback that is invoked when user presses "OK" button (or double-clicks a file). The file dialog is still open. If the callback returns `SystemFileChooser.CANCEL_OPTION`, then the file dialog stays open. If it returns `SystemFileChooser.APPROVE_OPTION` (or any value other than `SystemFileChooser.CANCEL_OPTION`), the file dialog is closed and the `show...Dialog()` methods return that value.

The callback has two parameters:

- `File[] selectedFiles` - one or more selected files
- `SystemFileChooser.ApproveContext context` - context object that provides additional methods

**WARNING:** Do not show a Swing dialog from within the callback. This will not work!

Instead, use `ApproveContext.showMessageDialog(int, String, String, int, String...)`, which shows a modal system message dialog as child of the file dialog.

```java
fc.setApproveCallback( (selectedFiles, context) -> {
    if( !selectedFiles[0].getName().startsWith( "blabla" ) ) {
        context.showMessageDialog( JOptionPane.WARNING_MESSAGE,
            "File name must start with 'blabla' :)", null, 0 );
        return SystemFileChooser.CANCEL_OPTION;
    }
    return SystemFileChooser.APPROVE_OPTION;
} );

```
This message dialog supports various icons (information, warning, error, question and plain), primary and secondary message texts, and multiple custom buttons:

### Platform specific properties

Platform properties allow using file dialog features that exist only on specific platforms. For supported platform properties see `SystemFileChooser.WINDOWS_`, `SystemFileChooser.MAC_` and `SystemFileChooser.LINUX_` constants. E.g.:

fc.putPlatformProperty( SystemFileChooser.WINDOWS_FILE_NAME_LABEL, "My filename label:" );
fc.putPlatformProperty( SystemFileChooser.MAC_TREATS_FILE_PACKAGES_AS_DIRECTORIES, true );

### State Storage

`SystemFileChooser` can remember last visited folder in some kind of application state storage. You need to implement interface `SystemFileChooser.StateStore` and set it once. Following example uses [`java.util.prefs.Preferences`](https://docs.oracle.com/en/java/javase/21/docs/api/java.prefs/java/util/prefs/Preferences.html):

```java
SystemFileChooser.setStateStore( new SystemFileChooser.StateStore() {
    private static final String KEY_PREFIX = "fileChooser.";

    private final Preferences state = Preferences.userRoot().node( "my-application" );

    @Override
    public String get( String key, String def ) {
        return state.get( KEY_PREFIX + key, def );
    }

    @Override
    public void put( String key, String value ) {
        if( value != null )
            state.put( KEY_PREFIX + key, value );
        else
            state.remove( KEY_PREFIX + key );
    }
} );
```

To avoid conflicts with other application state/preferences, it is a good idea to either add a prefix to the key (as `KEY_PREFIX` in above example), or use an independent node/folder for file chooser state storage (e.g. `Preferences.userRoot().node( "my-application/filechooser" )`).

### State Storage ID

By specifying an ID, an application can have different persisted states for different kinds of file dialogs within the application. E.g. Import/Export file dialogs could use a different ID then Open/Save file dialogs.

fc.setStateKeyPrefix( "im-export" );

## Limitations/incompatibilities compared to JFileChooser

- **Open File** and **Select Folder** dialogs always warn about not existing files/folders. The operating system shows a warning dialog to inform the user. It is not possible to customize that warning dialog. The file dialog stays open.
- **Save File** dialog always asks whether an existing file should be overwritten. The operating system shows a question dialog to ask the user whether he wants to overwrite the file or not. If user selects "Yes", the file dialog closes. If user selects "No", the file dialog stays open. It is not possible to customize that question dialog.
- **Save File** dialog does not support multi-selection.
- For selection mode `SystemFileChooser.DIRECTORIES_ONLY`, dialog type `SystemFileChooser.SAVE_DIALOG` is ignored. Operating system file dialogs support folder selection only in "Open" dialogs.
- [`JFileChooser.FILES_AND_DIRECTORIES`](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JFileChooser.html#FILES_AND_DIRECTORIES) is not supported.
- `SystemFileChooser.getSelectedFiles()` returns selected file also in single selection mode. [`JFileChooser.getSelectedFiles()`](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JFileChooser.html#getSelectedFiles\(\)) only in multi selection mode.
- Only file name extension filters (see `SystemFileChooser.FileNameExtensionFilter`) are supported.
- If adding choosable file filters and `SystemFileChooser.isAcceptAllFileFilterUsed()` is `true`, then the **All Files** filter is placed at the end of the combobox list (as usual in current operating systems) and the first choosable filter is selected by default. `JFileChooser`, on the other hand, adds **All Files** filter as first item and selects it by default. Use `fc.addChoosableFileFilter( fc.getAcceptAllFileFilter() )` to place **All Files** filter somewhere else.
- Accessory components are not supported.
- **macOS**: By default, the user can not navigate into file packages (e.g. applications). If needed, this can be enabled by setting platform property `SystemFileChooser.MAC_TREATS_FILE_PACKAGES_AS_DIRECTORIES` to `true`.
- If no "current directory" is specified, then `JFileChooser` always opens the users "Documents" folder (on Windows) or "Home" folder (on macOS and Linux).  
    `SystemFileChooser` does the same when first shown, but then remembers last visited folder (either in memory or in a `StateStore`) and re-uses that folder when `SystemFileChooser` is shown again.

## Disable

You can disable usage of operating system file dialogs by setting system property [`flatlaf.useSystemFileChooser`](https://www.formdev.com/flatlaf/system-properties#flatlaf.useSystemFileChooser) to `false`. Then `SystemFileChooser` uses `JFileChooser`.

E.g. in Java code:

```java
System.setProperty( "flatlaf.useSystemFileChooser", "false" );
// or
System.setProperty( FlatSystemProperties.USE_SYSTEM_FILE_CHOOSER, "false" );
```

Or at command line:

```
-Dflatlaf.useSystemFileChooser=false
```