# Pen Customizations

[Alexis Menard](https://github.com/darktears),
[Jimmy Huang](https://github.com/jimmy-huang)

## Related readings:
| Name | Link |
|------|------|
| Pointer Events | [Specification](https://w3c.github.io/pointerevents/) |
| Universal Stylus Initiative | [Website](https://universalstylus.org/)


## Motivation
In the past few years styluses has received several improvements to make them more capable. Historically only few properties have been available to web developers, the minimum to implement drawing capabilities. However they have become more capable beyond the basic features. Recently Microsoft proposed new [APIs](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/PenEvents/dev-design.md) to cover styluses who maintain a connection with the computer through other means than the digitizer.

Created few years ago, the Universal Stylus Initiative, [USI](https://universalstylus.org/) in short, defines a standard for interoperable communication between an active stylus and touch enabled devices. Many USI certified devices and styluses have been available on the market for quite some time. Basic functionalities are already working through the [Pointer Events](https://w3c.github.io/pointerevents/) but few new features brought by USI are not exposed to web developers. The proposal below will try to address this gap.

One of the main added functionality with USI is the ability to store preferences into the stylus itself. In other word a user can decide to store its favorite color which can then be retrieved by an application afterwards (whether or not it's on the same device). Supported attributes are colors, width and style.

## Proposed changes

We are proposing the following changes in the Pointer Events specification. Web developers are expected to consume stylus events through the PointerEvents API, we intend to extend it to support the new customizations. If the hardware does not support the capabilities the fields will be set to default value. This matches the behavior of the other fields when the hardware does not support it (e.g. tilt).


### Pointer Events
Here is a proposed IDL for the new PointerEvent interface:

```
[Exposed=Window]
interface PointerEvent : MouseEvent {
    constructor(DOMString type, optional PointerEventInit eventInitDict = {});
    readonly        attribute long        pointerId;
    readonly        attribute double      width;
    readonly        attribute double      height;
    readonly        attribute float       pressure;
    readonly        attribute float       tangentialPressure;
    readonly        attribute long        tiltX;
    readonly        attribute long        tiltY;
    readonly        attribute long        twist;
    readonly        attribute double      altitudeAngle;
    readonly        attribute double      azimuthAngle;
    readonly        attribute PenCustomizationsDetails penCustomizations;
    ##### NEW FIELD #####
    PenCustomizationsDetails penCustomizationsDetails;
    ##########################
    readonly        attribute DOMString   pointerType;
    readonly        attribute boolean     isPrimary;
    [SecureContext] sequence<PointerEvent> getCoalescedEvents();
    sequence<PointerEvent> getPredictedEvents();
};

interface PenCustomizationsDetails {
    Promise<DOMString>        getPreferredInkingColor();
    Promise<DOMString>        getPreferredInkingStyle();
    Promise<double>           getPreferredInkingStyle;
    Promise<DOMString> setPreferredInkingColor(DOMString preferredColor);
    Promise<DOMString> setPreferredInkingStyle(DOMString preferredType);
    Promise<undefined> setPreferredInkingWidth(double preferredWidth);
}

// Example of use.
function onPointerEvent(event) {
    let pointerEvent = event;
    let penCustomizations = pointerEvent.penCustomizationsDetails;
    penCustomizations.getPreferredInkingColor().then(function(color) { brushColor = color } );
}

writeColorButton.addEventListener('click', function(event) { this.onWriteColor(selectedColor, event)})

function onWriteColor(color, event) {
    let penCustomizations = event.penCustomizationsDetails;
    penCustomization.setPreferredInkingColor(color).then(function(return) {console.log('success'} );
}


```

For each customizations the promise will resolve if the customization was successfully written on the stylus. Most USI based stylus requires the stylus to be in proximity of the screen to successfully write data. It is strongly suggested to provide a UI in which the user will press with the stylus to write the customization.

Each methods, setPreferredInkingColor, setPreferredInkingStyle and setPreferredInkingWidth will resolve if the customization has been successfully saved. The promise will fail if writing to the pen failed. The promise will resolve with the written color on the memory. At this point USI styluses can only store 24 bits color and if the color passed in setPreferredInkingColor doesn't fit it will clamped to the closest possible color and returned. Alpha channel colors are also not supported by USI pens so in this case the value will be ignored.


