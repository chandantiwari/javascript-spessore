## Immutable Properties

Sometimes we want to share objects by reference for performance and space reasons, but we don't want them to be mutable. One motivation is when we want many objects to be able to share a common entity without worrying that one of them may inadvertently change the common entity.

JavaScript provides a way to make properties immutable:

    "use strict";

    var rentAmount = {};

    Object.defineProperty(rentAmount, 'dollars', {
      enumerable: true,
      writable: false,
      value: 420
    });

    Object.defineProperty(rentAmount, 'cents', {
      enumerable: true,
      writable: false,
      value: 0
    });

    rentAmount.dollars
      //=> 420

    rentAmount.dollars = 600;
      //=> 600

    rentAmount.dollars
      //=> 420

`Object.defineProperty` is a general-purpose method for providing fine-grained control over the properties of any object. When we make a property `enumerable`, it shows up whenever we list the object's properties or iterate over them. When we make it writable, assignments to the property change its value. If the property isn't writable, assignments are ignored.

When we want to define multiple properties, we can also write:

    var rentAmount = {};

    Object.defineProperties(rentAmount, {
      dollars: {
        enumerable: true,
        writable: false,
        value: 420
      },
      cents: {
        enumerable: true,
        writable: false,
        value: 0
      }
    });

    rentAmount.dollars
      //=> 420

    rentAmount.dollars = 600;
      //=> 600

    rentAmount.dollars
      //=> 420

While we can't make the entire object immutable, we can define the properties we want to be immutable. Naturally, we can generalize this:

    var allong = require('allong.es');
    var tap = allong.es.tap;

    function immutable (propertiesAndValues) {
      return tap({}, function (object) {
        for (var key in propertiesAndValues) {
          if (propertiesAndValues.hasOwnProperty(key)) {
            Object.defineProperty(object, key, {
              enumerable: true,
              writable: false,
              value: propertiesAndValues[key]
            });
          }
        }
      });
    }

    var rentAmount = immutable({
      dollars: 420,
      cents: 0
    });

    rentAmount.dollars
      //=> 420

    rentAmount.dollars = 600;
      //=> 600

    rentAmount.dollars
      //=> 420


### considerations

As a means for protecting our data structures from inadvertent modification, this "silent failure" isn't great, but it at least *localizes* the failure mode to the objects our code is trying to change, and not to objects that may be sharing an object we're trying to change.

But it does force us to test property assignments. Whenever you write some code like this:

    rentCheque.amount.dollars = 600;

You ought to write a test case that checks to see whether the cheque's dollar figure really changed. And now that you know that immutable properties are a JavaScript feature, you really need to check assignments whether you're using them or not. Who knows what changes might be made to your code in the future? Testing assignments ensures that you will catch any regressions that might be caused in the future.