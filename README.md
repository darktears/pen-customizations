# Pen Customizations

[Alexis Menard](https://github.com/darktears)

## Related readings:
| Name | Link |
|------|------|
| Pointer Events | [Specification](https://w3c.github.io/pointerevents/) |
| Universal Stylus Initiative | [Website](https://universalstylus.org/)


## Motivation
In the past few years styluses has received several improvements to make them more capable. Historically only few properties have been available to web developers, the minimum to implement drawing capabilities. However they have become more capable beyond the basic features. Recently Microsoft proposed new [APIs](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/PenEvents/dev-design.md) to cover styluses who maintain a connection with the computer through other means than the digitizer.

Created few years ago, the Universal Stylus Initiative, [USI](https://universalstylus.org/) in short, defines a standard for interoperable communication between an active stylus and touch enabled devices. Many USI certified devices and styluses have been available on the market for quite some time. Basic functionalities are already working through the [Pointer Events](https://w3c.github.io/pointerevents/) but few new features brought by USI are not exposed to web developers. The proposal below will try to address this gap.

One of the main added functionality with USI is the ability to store preferences into the stylus itself. In other word a user can decide to store its favorite color which can then be retrieved by an application afterwards (whether or not it's on the same device). Supported attributes are colors, width and style.

We identified few main use cases for this API:
* Ability to use multiple stylus previously setup (meaning custom customizations have been stored on them) when drawing. The artist/designer can then swap quickly between them and the drawing application can then automatically select the right drawing tool/width/style automatically. This will mimic the behavior of real life a pencil box.
* Ability to carry over customizations from machine to machine.
* Some USI stylus have a sensor to read color from the real world. When used the sensor will then write the color in the memory of the stylus. Using the proposed API the drawing application can then retrieve the said color and set it into the editor. This provides a "real life" color picker.

You can see the use cases in action here : [video](https://www.youtube.com/watch?v=t_wQm3dPpqI).

## Proposed changes

We are proposing the following changes in the Pointer Events specification. Web developers are expected to consume stylus events through the PointerEvents API, we intend to extend it to support the new customizations. If the hardware does not support the capabilities the added field will be set to null. This matches the behavior of the other fields when the hardware does not support it (e.g. tilt).

### Pointer Events
Here are few snippets of code to work with the proposed API.

```
// Fetching the pen customizations.
function onPointerEvent(event) {
    let pointerEvent = event;
    let penCustomizations = pointerEvent.penCustomizationsDetails;
    penCustomizations.getPreferredInkingColor().then(function(color) { brushColor = color } );
}

writeColorButton.addEventListener('onpointerdown', function(event) { this.onWriteColor(selectedColor, event)})

function onWriteColor(color, event) {
    let penCustomizations = event.penCustomizationsDetails;
    penCustomization.setPreferredInkingColor(color).then(function(return) {console.log('success'} );
}

```

Because pen customizations are only available for pens, the ```penCustomizationsDetails``` field will be set to null if :

* The platform doesn't support pen customizations (for e.g. macOS, iOS, Android)
* The Pointer Event is not of type ```pen```

All methods in ```penCustomizationsDetails``` will resolve if the UA is able to retrieve/store the attributes from the stylus. In case it's not possible the UA will fail the promise with an error.


Most USI based stylus requires the stylus to be in proximity of the screen to successfully write/retrieve data. It is strongly suggested to provide a UI in which the user will press with the stylus to write/read the customizations. For that same reason the ```penCustomizationsDetails``` is not defined on pointer up events because it will likely lead to a poor user experience : by the time the pointer up event is delivered the stylus is likely out of range. 


Each methods, setPreferredInkingColor, setPreferredInkingStyle and setPreferredInkingWidth will resolve if the customization has been successfully saved.


Note on color support: It is up to the UA to determine what color capabilities are supported from the hardware and clamp the color if required. For example USI styluses can only store 24 bits color and if the color passed in setPreferredInkingColor doesn't fit it will clamped to the closest possible color and returned. Alpha channel colors are also not supported by USI pens so in this case the value will be ignored.
