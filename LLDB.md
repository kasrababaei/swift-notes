# LLDB

A few useful commands that can be used in the debugger console:

```swift
p (BOOL)[0x7fc5bb41a150 isHidden]
p (void)[0x7fc5bb412840 setHidden:YES]
po [0x7f8c7b002b30 setText:@”Gold”]
po (NSAttributedString*)[0x7f7f5802a000 attributedText] // It casts attributedText to NSAttributedStrings as well
po (void)[0x7f93a00e8000 setTextColor: [UIColor redColor]]
po (void)[0x7fdc33452010 setTextAlignment:1]
po (void)[0x7fe02e458060 setBackgroundColor:[UIColor redColor]]
p (UIEdgeInsets)[0x7f9fe679d790 layoutMargins]
p (CGRect)[0x7f8ac4c76b50 frame]
po (void)[0x7fa20c03c290 setText: @"Hello, World!"] // Change text of a UILabel
po [0x7f8887686490 recursiveDescription] // Outputs view hierarchy 
po (void)[0x7f88876b80f0 setFrame:(CGRect){16,0,343,20}] // Updates the frame of a UIView 
po ((UIView*) 0x7fb5f7f332c0).frame.origin
p (int)[[0x7fe2c6978da0 arrangedSubviews] count]
expression let $variable = animal.optionalName
po (void)[0x7f994dea7820 setContentCompressionResistancePriority:UILayoutPriorityRequired forAxis:0]
po (void)[0x7f994dea7820 setContentCompressionResistancePriority:999 forAxis:0]
po [(UITextView*)0x14501b400 setText:@"Hello, World!"]
po [(UIView*)0x131e2c2a0 layoutIfNeeded]
e [(UIStackView*) 0x14870bd70 setAlignment: UIStackViewAlignmentFill] // Need to do `e @import UIKit` first
p (void)[0x136f17410 setNumberOfLines:2]
p (UIEdgeInsets)[[0x16203ac00 view] safeAreaInsets]
```
