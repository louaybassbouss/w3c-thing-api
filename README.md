# W3C Thing API

The Thing API is an experimental API for Discovery, Provisioning and Control of Things in a [Web of Things](http://www.w3.org/WoT/). A thing may compromise of multiple sensors and/or actuators and a corresponding [Thing Description](http://www.w3.org/WoT/) that describes the properties, actions, events and metadata of the corresponding thing. The metadata includes also a set of protocol bindings. 

# Interfaces

## Interface `ThingRequest`

```webidl
[Constructor(ThingFilter filter)]
interface ThingRequest {
    Promise<sequence<Thing>> start();
};
```

## Interface `ThingFilter`

```webidl
dictionary ThingFilter {
    attribute DOMString? type;
    attribute ThingProximity? proximity;
    attribute DOMString? id;
    attribute DOMString? server;
};
```

## Interface `ThingProximity`
                                                                                                                 
```webidl
enum ThingProximity { 
    "local", 
    "nearby", 
    "remote" 
};
```

## Interface `Thing`

```webidl
interface Thing: EventTarget {
    readonly attribute DOMString id;
    readonly attribute DOMString type;
    readonly attribute DOMString name;
    readonly attribute DOMString manufacturer;
    readonly attribute boolean reachable;
    attribute EventHandler onreachabilitychange;
    Promise<any> callAction(DOMString actionName, any parameter);
    Promise<any> setProperty(DOMString propertyName, any newValue);
    Promise<any> getProperty(DOMString propertyName);
    void addListener(DOMString eventName, ThingEventListener listener);
    void removeListener(DOMString eventName, ThingEventListener listener);
    void removeAllListeners(DOMString eventName);
}          
callback ThingEventListener = void (ThingEvent event);
```

## Interface `ThingEvent`

```webidl
interface ThingEvent {
    readonly attribute DOMString name;
    readonly attribute any value;
    readonly attribute Thing source;
}
```

# Example

```javascript
var filter = {
    type: "http://example.org#foo",
    proximity: "nearby"
};
var request = new ThingRequest(filter);
request.start().then(function(things){
    var thing = things[0];
    if(thing){
        // get thing basic information
        console.log("id: ", thing.id);
        console.log("name: ", thing.name);
        console.log("type: ", thing.type);
        console.log("manufacturer: ", thing.manufacturer);
        console.log("reachable: ", thing.reachable);
        // monitor reachability of the thing
        thing.onreachabilitychange = function(){
            console.log("reachability changed to ", this.reachable);
            // If the thing is not reachable, then the operations callAction(), getProperty() and setProperty() will fail and the promise will be rejected with a corresponding error. The operations addListener(), removeListener() and removeAllListeners() will not fail, but events will be fired when the thing is reachable again.
        };
        // Call an action
        var input = ...;
        thing.callAction("myAction",input).then(function(output){
            console.log("Result of myAction()",output);
        }).catch(function(err){
            console.error("Error on call action",err);
        });
        // get and set property
        thing.getProperty("myProp").then(function(value){
            console.log("Value of myProp ",value);
            var newValue = ...;
            return thing.setProperty("myProp", newValue);
        }).then(function(newValue){
            console.log("Value of myProp is now",newValue);
        }).catch(function(err){
            console.error("Error on get or set property myProp",err);
        });
        // add and remove thing event listener
        var myListener;
        thing.addListener("myEvent",myListener=function(evt){
            console.log("receive event ",name,"from thing",evt.source.name,"with value",evt.value);           
        });
        thing.removeListener("myEvent",myListener);
        thing.removeAllListeners("myEvent");
    };
}).catch(function(err){
    //TODO: handle error
});
```