# Cheat Sheet de Frida


### Instalación:
```sh
pip install frida
pip install frida-tools

### Comprobar instalación:
frida --version

### Conexión a Dispositivos:
adb devices
frida-ps -U

### Conectar a un dispositivo remoto:
frida-ps -H <remote-ip>

### Listar procesos en el dispositivo:
frida-ps -U

### Listar aplicaciones instaladas:
frida-ls-devices

### Inyectar un script en una aplicación:
frida -U -f com.example.app -l script.js --no-pause

### Script básico (script.js):
console.log("Script inyectado correctamente");

### Interceptar una función en una biblioteca nativa:
Interceptor.attach(Module.findExportByName("libexample.so", "example_function"), {
    onEnter: function (args) {
        console.log("example_function llamada con argumentos:", args[0].toInt32());
    },
    onLeave: function (retval) {
        console.log("example_function retornó:", retval.toInt32());
    }
});

### Enumerar clases y métodos:
Java.perform(function () {
    Java.enumerateLoadedClasses({
        onMatch: function (className) {
            console.log(className);
        },
        onComplete: function () {
            console.log("Enumeración completa");
        }
    });
});

### Interceptar un método de Java:
Java.perform(function () {
    var MainActivity = Java.use("com.example.app.MainActivity");
    MainActivity.onCreate.overload('android.os.Bundle').implementation = function (bundle) {
        console.log("onCreate interceptado");
        this.onCreate(bundle);
    };
});

### Modificar el retorno de una función:
Interceptor.attach(Module.findExportByName("libexample.so", "example_function"), {
    onLeave: function (retval) {
        retval.replace(42);
    }
});

### Modificar el retorno de un método de Java:
Java.perform(function () {
    var MainActivity = Java.use("com.example.app.MainActivity");
    MainActivity.getString.implementation = function () {
        return "String modificada";
    };
});

### Enviar y recibir mensajes entre Frida y el script:
// En el script (script.js)
rpc.exports = {
    hello: function () {
        return "Hello from Frida";
    }
};

// Desde la consola Frida
frida -U -f com.example.app --no-pause

import frida
def on_message(message, data):
    print(message)
session = frida.get_usb_device().attach("com.example.app")
script = session.create_script(open("script.js").read())
script.on("message", on_message)
script.load()
print(script.exports.hello())

### Utilizar Interceptor.replace para cambiar completamente una función:
var newFunction = new NativeCallback(function (arg1, arg2) {
    console.log("newFunction llamada con", arg1, arg2);
    return 0;
}, 'int', ['int', 'int']);

Interceptor.replace(Module.findExportByName("libexample.so", "example_function"), newFunction);


