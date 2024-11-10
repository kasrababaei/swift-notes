# Accessibility

Accessibility features help a wide range of people interact with their devices. By creating your app with accessibility in mind, you make it possible for everyone to enjoy your app.

Apple has wide range of accessibility APIs:

- Vision: A person may be blind or color blind, or have a vision challenge that makes focusing difficult.
- Speech: A person may have a speech disability or prefer to connect without using their voice.
- Mobility: A person with reduced mobility may have difficulty holding a device or tapping the interface.
- Cognitive: A person may have difficulty remembering a sequence of steps, or they may find an overly complex user interface difficult to process and manage.
- Hearing: A person may be deaf, have partial hearing loss, or have difficulty hearing sounds within a certain range.

People can personalize their devices by choosing the accessibility features and assistive technologies that give them the best user experience. Apple has a wide range of assistive technologies:

- VoiceOver: A gesture-based screen reader that provides an auditory description of the content onscreen.
- Voice Control: An interface for navigating a device using voice commands to tap, swipe, type, and more.
- Switch Control: An interface for navigating a device with a variety of adaptive switch hardware, wireless game controllers, or sounds such as a click or a pop.
- Assistive Access: A mode that tailors the iOS and iPadOS experience for people with cognitive disabilities.

## Vision

In WWDC21, the [Bring Accessibility to Your Chart](https://developer.apple.com/videos/play/wwdc2021/10122) session showed how to make charts accessible in your app to people with vision impairments through audio graph and sonified data.

By overriding `accessibilityElements: [Any]?` and providing elements of `UIAccessibilityElement` and setting `accessibilityFrameInContainerSpace` to each point in the chart, it's possible to make VoiceOver announce the `accessibilityLabel` and `accessibilityValue` as user taps on different areas of the graph.

Also, you'll want to do is make the chart a container. This will help VoiceOver group your chart elements correctly and aid in navigation.

To make your chart a container, simply override accessibilityContainerType on your ChartView and return the semanticGroup container type.
