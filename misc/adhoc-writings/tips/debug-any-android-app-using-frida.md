# Debug any Android app using Frida

What will you do when examining a published Android app from the Playstore. Yes, you can obtain their APK and just decompile/reverse it to get the source code. But sometimes you may be too lazy to read the code, especially in cases where they are heavily obfuscated.

Do you remember what those cool-geeky-hackerz-leets said when **static analysis** did not work anymore? Do **dynamic analysis**!

We can use Frida to help us doing this dynamic analysis. Let's see this code:

```text
Java.perform(function() {
  const targetClass = Java.use('com.example.TargetClass');
  targetClass.targetMethod.implementation = function() {
    const argumentsJson = JSON.stringify(arguments, null, 2);
    const returnValue = targetClass.targetMethod.apply(this, arguments);

    console.log('TARGETED_METHOD_CALLED');
    console.log('ARGUMENTS:', argumentsJson);
    console.log('RETURN_VALUE:', returnValue);

    return returnValue;
  }
});
```

Well, actually the above script is more about outputting the calling method's arguments values and the resulting return value.

