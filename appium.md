# Selector Strategies

Strategy |	Description
------------ | -------------
Accessibility ID |	Read a unique identifier for a UI element. For XCUITest it is the element's accessibility-id attribute. For Android it is the element's content-desc attribute.
Class name |	For IOS it is the full name of the XCUI element and begins with XCUIElementType. For Android it is the full name of the UIAutomator2 class (e.g.: android.widget.TextView)
ID |	Native element identifier. **resource-id** for android; **name** for iOS.
Name |	Name of element
XPath |	Search the app XML source using xpath (not recommended, has performance issues)
Image |	Locate an element by matching it with a base 64 encoded image file
