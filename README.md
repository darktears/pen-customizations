# Stylus Customizations

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

We are proposing the following changes in the Pointer Events specification. Web developers are expected to consume stylus events through the PointerEvents API, we intend to extend it to support the new preferences. If the hardware does not support the capabilities the fields will be set to ```null```. This matches the behavior of the other fields when the hardware does not support it (e.g. tilt).


### Pointer Events
Here is a proposed IDL for the new PointerEvent interface:

```
dictionary PointerEventInit : MouseEventInit {
    long        pointerId = 0;
    double      width = 1;
    double      height = 1;
    float       pressure = 0;
    float       tangentialPressure = 0;
    long        tiltX;
    long        tiltY;
    long        twist = 0;
    double      altitudeAngle;
    double      azimuthAngle;
    ##### NEW PROPERTIES #####
    DOMString   preferredColor;
    DOMString   preferredType;
    double      preferredWidth;
    ##########################
    DOMString   pointerType = "";
    boolean     isPrimary = false;
    sequence<PointerEvent> coalescedEvents = [];
    sequence<PointerEvent> predictedEvents = [];
};

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
    ##### NEW PROPERTIES #####
    readonly        attribute DOMString   preferredColor;
    readonly        attribute DOMString   preferredType;
    readonly        attribute double      preferredWidth;
    ##########################
    readonly        attribute DOMString   pointerType;
    readonly        attribute boolean     isPrimary;
    [SecureContext] sequence<PointerEvent> getCoalescedEvents();
    sequence<PointerEvent> getPredictedEvents();
};
```

### Setting preferences in the stylus' memory

In order to set preferences on the stylus memory we need a new interface to send the new values to the stylus.

```
partial interface Navigator {
    sequence<StylusCustomizations?> getStylusCustomizations();
};
```

The returned array will be ```null``` if the user is not using styluses supporting customizations.

```
[Exposed=Window, SecureContext]
interface StylusCustomizations {
  readonly attribute long index;
  Promise<DOMString> setPreferredColor(DOMString preferredColor);
  Promise<DOMString> setPreferredType(DOMString preferredType);
  Promise<DOMString> setPreferredWidth(double preferredWidth);
  Promise<WrittenCustomizations> setCustomizations(StylusCustomizations
                                       customizations = {});
};
```

For each customization the promise will resolve if the customization was successfully written on the stylus. Most USI based stylus requires the stylus to be in proximity of the screen to successfully write data. It is strongly suggested to provide a UI in which the user will press with the stylus to write the customization.

Each methods, setPreferredColor, setPreferredType and setPreferredWidth will resolve if the customization has been successfully saved. The promise will fail if writing to the stylus failed. The promise will resolve with the written color on the memory. At this point USI styluses can only store 24 bits color and if the color passed in setPreferredColor doesn't fit it will clamped to the closest possible color and returned.

The setCustomizations methods allows you to set multiple values in a single method call.

Newer hardware can support multiple stylus at the same time therefore an index is required to identify the stylus to be targeted. The index of the stylus in the Navigator. When multiple styluses are connected to a user agent, indices MUST be assigned on a first-come, first-serve basis, starting at zero. If a stylus is going out of range, previously assigned indices MUST NOT be reassigned to styluses that continue to be connected. However, if a stylus is disconnected, and subsequently the same or a different stylus is then connected, the lowest previously used index MUST be reused.

