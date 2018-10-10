# DelphiUIAutomation
Delphi classes that wrap the [MS UIAutomation library](https://msdn.microsoft.com/en-us/library/vstudio/ms753388(v=vs.100).aspx).

DelphiUIAutomation is a framework for automating rich client applications based on Win32 (and specifically tested with Delphi XE5 and above). It is written in Delphi and it requires no use of scripting languages.

Tests and automation programs using DelphiUIAutomation can be written with Delphi and used in any testing framework available to Delphi.

It provides a consistent object-oriented API, hiding the complexity of Microsoft's UIAutomation library and windows messages.

## Getting started

The DelphiUIAutomation library is a wrapper for the UIAutomationClient library, which has been extracted into the UIAutomationClient_TLB source file. As the generated code is large and complex, this has been wrapped up in a number of units, each providing classes that encapsulate part of this library (together with other utility methods).

### Initialise the libray

```pascal
  TUIAuto.CreateUIAuto;   // Initialise the library
```

### Launching an application

The TAutomationApplication class provides functionality to start and attach to an application. There are 3 class methods provided to do this.

* Launch - this will launch the application supplied, and pass in any supplied arguments
* Attach - this will attach to an already launched application, based on the executable name
* LaunchOrAttach - this will either attach to an already launched application, or launch the application.

The snippet below will check whether the Project1.exe is running, and attach if it is, or it will start the application, and then attach to it.

```pascal
Application := TAutomationApplication.LaunchOrAttach(
      'democlient\Win32\Debug\Project1.exe',
      ''
```

When attached to an application, it is possible to call further methods on it

* Kill - Will kill the process associated with the application
* WaitWhileBusy - Waits for the application to be idle, this has an optional timeout (in milliseconds) associated with it
* SaveScreenshot - Takes a screenshot of the application, and then saves this to a file.

### Getting hold of a window

To get a 'desktop' window (i.e. one that appears in the Windows tasks bar), then the TAutomationDesktop class provides a class function that returns a TAutomationWindow object.

```pascal
var
  notepad : IAutomationWindow;
  ...
  
  notepad := TAutomationDesktop.GetDesktopWindow('Untitled - Notepad');
  notepad.Focus;
```

This will find (it is there) a window that has the given title, and set focus to it. This window is independant of the overalll application, and might not even be associated with the same application that is being automated.

To get an 'application' window, i.e. one associated with another window, first the parent window must be found, and then the target child can be found using the ''Window'' method. In the example below, the child window 'Security' of the notepad window is searched for.

```pascal
var
  security : IAutomationWindow
  ...
  
  security := notepad.Window('Security');
```

### Finding a control

Each control contained in a window can be identified by the index of that control OR sometimes (this depends on the control type) by the text associated with it. For example, in order to get the textbox associated with the connection window (and assuming that it is the 1st Edit box on the window), the following code will find the editbox, and change the text to be USER1.

```pascal
var
  user : IAutomationEditBox;

  user := connect.GetEditBoxByIndex(0);
  user.Text := 'USER1';
```

### Invoking actions on controls

In order to click the 'OK' button associated with the connection window, it can be found by the text associated with the button, the following code will find the button, and call the click event.

```pascal
var
  btnOK : IAutomationButton;
  ...
  
  btnOK := connect.GetButton('OK');
  btnOk.Click;
```

# Current supported controls

The currently supported controls are ...

* TButton
* TCheckBox
* TComboBox
* TEditBox
* TRadioButton
* TStatusBar
* TStringGrid (using an extended TStringGrid control that implements UIAutomation patterns)
* TPageControl
* TTab
* TTextBox
* TTreeView and TTreeViewItem


[More details, and the status of currently supported controls](https://github.com/jhc-systems/DelphiUIAutomation/wiki/CurrentSupportedControls)

# Added Automation Support for Controls

Many Delphi controls do not implement the automatin interfaces in the same manner as Visual Studio does in WPF, so that the Automation ID and Name are not 'properly' populated, so the controls can only be found by knowing their position within the tree, and cannot be found via the name or ID. The controls below extend the basic controls to export these values, amongst other properties.

## TEdit & TCombobox

The [controls sub-project](https://github.com/jhc-systems/DelphiUIAutomation/tree/master/controls) extends the automation properties of the TEdit and TComboBox, to simulate the way that WPF populates the Automation ID and the name with the NAME of the actual control, not a random value.

## TStringGrid

The [controls sub-project](https://github.com/jhc-systems/DelphiUIAutomation/tree/master/controls) allows the automation of some of the elements of the TStringGrid. It extends the control to allow it to interact with the MS-UIAutomation libraries.

```pascal
var
  grid : IAutomationStringGrid;
  items : TObjectList<TAutomationStringGridItem>;
  item : TAutomationStringGridItem;
  item1 : IAutomationStringGridItem;
  ...

  // Get the first string grid associated with the window
  grid := enquiry.GetStringGridByIndex(0);

  // Show what the value is (i.e. the contents of the selected cell)
  writeln ('Value is ' + grid.Value);

  // Get the cell at 3,3 and shows it's value
  writeln ('Item @ 3-3 is ' +grid.GetItem(3,3).Name);

  // Get the selected cell
  item := grid.Selected;

  // Show the value of the selected cell (should be the same as the Grid's value
  writeln ('Selected is ' + item.Name);

  // Get the list of column headers (i.e. first fixed row)
  write ('Column Headers : ');
  items := grid.ColumnHeaders;

  for item in items do
  begin
    writeln (item.Name);
  end;

  // Select the item at 2,4
  item1 := grid.GetItem(2,4);

  // Show that selection has changed.
  writeln ('Selected is ' + grid.Selected.Name);

```

## TTreeView and TTreeViewItem
```pascal
  // Get the 0th treeview
  tv1 := enquiry.getTreeViewByIndex(0);

  // Get the item with the following text
  tvi := tv1.GetItem('Sub-SubItem');

  // Select the item
  tvi.select;
```

## Navigating to specific elements in the StringGrid and right-clicking

As the automation does not expose the cells fully (as they do not technically exist in the TStringGrid), it is necessary to do the following ..

```pascal
  // Get the grid item
  item := grid.GetItem(3,3);

  // Select it
  item.Select;
  
  // Create a mouse to move the pointer
  mouse := TAutomationMouse.Create;
  
  // Get the bounding rectangle of the item (this is relative to the grid)
  itemRect := item.BoundingRectangle;
  
  // Get the overall grid bounding rectangle
  gridRect := grid.BoundingRectangle;
  
  // Move to the correct location, offsetting to make sure the mouse point is inside the cells itself
  mouse.Location := TPoint.Create(gridRect.left + itemRect.left +15, gridRect.Top + itemRect.top +15);
  mouse.LeftClick;
  mouse.RightClick;
```

# Contributors
* [Mark Humphreys](https://github.com/markhumphreysjhc)
* Robert Deutschmann - Example Howto and AutomatedButton

# License
Apache Version 2.0 Copyright (C) 2015

See license.txt for details.

# See also
* [Microsoft Accessibility](https://msdn.microsoft.com/en-us/library/vstudio/ms753388(v=vs.100).aspx)
* [UIAutomation for Powershell](http://uiautomation.codeplex.com/documentation)
* [TestStack.White](https://github.com/TestStack/White)
* [UI Automation - A wrapper around Microsoft's UI Automation libraries.](https://github.com/vijayakumarsuraj/UIAutomation)

