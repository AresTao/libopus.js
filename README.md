# libopus.js

This is a port of [libopus] (1.1.2) to pure JavaScript using [Emscripten] for use in a browser environment.

If you do not plan to use your module in a browser-like environment, you should probably use [node-opus] instead which will almost certainly provide better performance at the cost of portability.

### Usage

JavaScript wrappers are provided for the most common usage scenarios.

Be aware that opus is a stateful codec, therefore one and only one Encoder/Decoder should be used for one and only one stream of audio data. Packet loss also has to be signalled to the decoder.

Encoding raw PCM samples:
```javascript
var Encoder = require('libopus.js').Encoder;
var enc = new Encoder({ rate: 48000, channels: 1 });
// Accepts either 16 bit signed integers (more space efficient)
var result = enc.encode(Int16Array.from([0, 256, 512, 256, 0, -256, -512));
// or 32 bit floating point numbers (used by the Web Audio API)
var result = enc.encode(Float32Array.from([0, 0.25, 0.5, 0.25]));
// result is a nodejs Buffer
someStream.write(result);
```

The encoder can also be used with node.js streams:
```javascript
var Encoder = require('libopus.js').Encoder;
var enc = new Encoder();
// A stream can only process input in either the Int16 format or the Float32 format
var encStream = enc.stream('Int16');
// The stream can be used directly
encStream.write(Buffer.from([0, 0, 0, 1, 0, 2]));
var result = encStream.read();
// or just like any other node stream
someRawInput.pipe(encStream).pipe(someOutputStream);
```

Decoding compressed opus packets:
```javascript
var Decoder = require('libopus.js').Decoder;
var dec = new Decoder({ rate: 48000, channels: 1 });
// Decode to Int16Array (more space efficient)
var result = dec.decodeInt16(Buffer.from(input));
// or to Float32Array (used by the Web Audio API)
var result = dec.decodeFloat32(Buffer.from(input));
// Signaling a lost packet
var result = dec.decodeInt16(null);
```

The decoder can also be used with node.js streams:
```javascript
var Decoder = require('libopus.js').Decoder;
var dec = new Decoder();
// A stream can only produce output in either the Int16 format or the Float32 format
var decStream = dec.stream('Int16');
// The stream can be used directly
decStream.write(Buffer.from(input));
var result = encStream.read();
// or just like any other node stream
someEncodedInput.pipe(decStream).pipe(someOutputStream);
// Signaling a lost packet in a stream is done by sending an empty Buffer
encStream.write(Buffer.alloc(0));
```

The raw libopus module as generated by emscripten can also be accessed (this is dangerous if you do not know what you are doing):
```javascript
var libopus = require('libopus.js').libopus;
var version = libopus.Pointer_stringify(libopus._opus_get_version_string());
var mem = libopus._malloc(42);
libopus._free(mem);
```

The Repacketizer, the Opus Multistream API, Opus Custom and some other parts have not been exported.

### Building from source

Prebuilt libopus binaries are available in `build/`.
Building these yourself is rather simple (assuming you have common build tools already installed):

1. [Install Emscripten]
2. Run `make clean`
3. Run `make`

### License

The full license texts are available in `LICENSE.md`.

libopus.js itself uses the MIT license.

The Makefile has been extracted from [Chris Rudmin's fork of Recorderjs] which also uses the MIT license.

The native opus library is licensed under a three-clause BSD license. See the
[Opus Licensing] page for more details. Therefore the generated libopus.js file is as well.

[libopus]: https://opus-codec.org/
[Emscripten]: http://emscripten.org/
[node-opus]: https://github.com/Rantanen/node-opus
[Chris Rudmin's fork of Recorderjs]: https://github.com/chris-rudmin/Recorderjs
[Opus Licensing]: https://www.opus-codec.org/license/
[Install Emscripten]: http://kripken.github.io/emscripten-site/docs/getting_started/downloads.html