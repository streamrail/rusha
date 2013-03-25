# Rusha
*A high-performance pure-javascript SHA1 implementation suitable for large binary data.*

## Prologue: The Sad State of Javascript SHA1 implementations

When we started experimenting with alternative upload technologies at [doctape](http://doctape.com) that required creating SHA1 hashes of the data locally on the client, it quickly became obvious that there were no performant pure-js implementations of SHA1 that worked correctly on binary data.

Jeff Motts [CryptoJS](http://code.google.com/p/crypto-js/), Brian Tureks [jsSHA](http://caligatio.github.com/jsSHA/) and Tim Caswells [Cifre](http://github.com/openpeer/cifre) were all hash functions that worked correctly on ASCII strings of a small size, but didn't scale to large data and/or didn't work correctly with binary data.

By modifying Paul Johnstons [sha1.js](http://pajhome.org.uk/crypt/md5/sha1.html) slightly, it worked correctly on binary data but was unforunately very slow, especially on V8. So a few days were invested on my side to implement a Johnston-inspired SHA1 hashing function with a heavy focus on performance.

The result of this process is Rusha, a SHA1 hash function that works flawlessly on large amounts binary data, such as binary strings or ArrayBuffers returned by the HTML5 File API.

## Installing

### Node.JS

There is really no point in doing this, since Node.JS already has a wonderful `crypto` module that is leveraging low-level hardware instructions to perform really nice. Your can see the comparison below in the benchmarks.

If you still want to do this, anyhow, just `require()` the `rusha.js` file, follow the instructions on _Using the Rusha Object_.

### Browser

It is highly recommended to run CPU-intensive tasks in a [Web Worker](http://developer.mozilla.org/en-US/docs/DOM/Using_web_workers). To do so, just start a worker with `var worker = new Worker('rusha.js')` and start sending it jobs. Follow the instructions on _Using the Rusha Worker_.

If you can't, for any reason, use Web Workers, include the `rusha.js` file in a `<script>` tag and follow the instructions on _Using the Rusha Object_.

## Using the Rusha Object

Your instantiate a new Rusha object by doing `var r = new Rusha(optionalSizeHint)`. When created, it provides the following methods:

- `Rusha#digestFromString(s)`: Create a hex digest from a binary `String`. A binary string is expected to only contain characters whose charCode < 256.
- `Rusha#digestFromBuffer(b)`: Create a hex digest from a `Buffer` or `Array`. Both are expected to only contain elements < 256.
- `Rusha#digestFromArrayBuffer(a)`: Create a hex digest from an `ArrayBuffer` object.

## Using the Rusha Worker

You can send your instance of the web worker messages in the format `{id: jobid, data: dataobject}`. The worker then sends back a message in the format `{id: jobid, hash: hash}`, were jobid is the id of the job previously received and hash is the hash of the dataobject you passed, be it a `Blob`, `Array`, `Buffer`, `ArrayBuffer` or `String`.

## Benchmarks

Tested were my Rusha implementation, the sha1.js implementation by [P. A. Johnston](http://pajhome.org.uk/crypt/md5/sha1.html) and the Node.JS native implementation.

All tests were performed on a MacBook Air 1.7 GHz Intel Core i5 and 4 GB 1333 MHz DDR3.

Node (V8) 0.8.18:

	Benchmarking 4096 bytes ...
	Native   emitted b180fa62481c53fcf86c21e7f6e319d4555ebd7a in 0 milliseconds
	Rusha    emitted b180fa62481c53fcf86c21e7f6e319d4555ebd7a in 5 milliseconds
	Johnst.  emitted b180fa62481c53fcf86c21e7f6e319d4555ebd7a in 3 milliseconds
	Benchmarking 1048576 bytes ...
	Native   emitted 6d7471eace53611faef8541b78d8130558a34070 in 4 milliseconds
	Rusha    emitted 6d7471eace53611faef8541b78d8130558a34070 in 29 milliseconds
	Johnst.  emitted 6d7471eace53611faef8541b78d8130558a34070 in 100 milliseconds
	Benchmarking 4194304 bytes ...
	Native   emitted bf8ed89073ea76ff9b9ac44d203f3c421669dd37 in 17 milliseconds
	Rusha    emitted bf8ed89073ea76ff9b9ac44d203f3c421669dd37 in 110 milliseconds
	Johnst.  emitted bf8ed89073ea76ff9b9ac44d203f3c421669dd37 in 431 milliseconds
	Benchmarking 8388608 bytes ...
	Native   emitted c9bbff50949da456fcf4d78093510f6e599e1568 in 37 milliseconds
	Rusha    emitted c9bbff50949da456fcf4d78093510f6e599e1568 in 220 milliseconds
	Johnst.  emitted c9bbff50949da456fcf4d78093510f6e599e1568 in 686 milliseconds

Chrome (V8) 25.0:

	Benchmarking 4096 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 67e1504a88a87adc7e6498872c8296030c501f52 in 5 milliseconds
	Johnst.  emitted 67e1504a88a87adc7e6498872c8296030c501f52 in 9 milliseconds
	Benchmarking 1048576 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 1339a85453885ee434a04a47f772d74b49ba6bac in 65 milliseconds
	Johnst.  emitted 1339a85453885ee434a04a47f772d74b49ba6bac in 261 milliseconds 	Benchmarking 4194304 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 4861c387e4c854f86e11e9e7f5565e5864235aa7 in 228 milliseconds
	Johnst.  emitted 4861c387e4c854f86e11e9e7f5565e5864235aa7 in 844 milliseconds
	Benchmarking 8388608 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 2f04698c8509c96e7fe15daa4e2385d1a1b1f0f0 in 462 milliseconds
	Johnst.  emitted 2f04698c8509c96e7fe15daa4e2385d1a1b1f0f0 in 1929 milliseconds

Safari 6.0.2:

	Benchmarking 4096 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted c6f613ccd8b9a71945dccdd9268abe92f9796690 in 4 milliseconds
	Johnst.  emitted c6f613ccd8b9a71945dccdd9268abe92f9796690 in 3 milliseconds
	Benchmarking 1048576 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 134ad263901ee070bd2ff7848294fd898e6d834c in 32 milliseconds
	Johnst.  emitted 134ad263901ee070bd2ff7848294fd898e6d834c in 83 milliseconds
	Benchmarking 4194304 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 4e6a71b9e1d2d00ba73051198ed39397dad3db4b in 104 milliseconds
	Johnst.  emitted 4e6a71b9e1d2d00ba73051198ed39397dad3db4b in 331 milliseconds
	Benchmarking 8388608 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted f0f5d94371727bf4c413dadb722eee4af971931e in 281 milliseconds
	Johnst.  emitted f0f5d94371727bf4c413dadb722eee4af971931e in 774 milliseconds

Firefox Aurora 21.0a2:

	Benchmarking 4096 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 2b06ba1f166c14bf66c21fc3c70fb9ca0e916cd3 in 6 milliseconds
	Johnst.  emitted 2b06ba1f166c14bf66c21fc3c70fb9ca0e916cd3 in 4 milliseconds
	Benchmarking 1048576 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 82eecaa400182aa0930c7b0b096583c3a69f300e in 22 milliseconds
	Johnst.  emitted 82eecaa400182aa0930c7b0b096583c3a69f300e in 55 milliseconds
	Benchmarking 4194304 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 9ad3fd23dd6d53466f4ee37a60c5b20e0d07fc4a in 71 milliseconds
	Johnst.  emitted 9ad3fd23dd6d53466f4ee37a60c5b20e0d07fc4a in 211 milliseconds
	Benchmarking 8388608 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted ae5472ffd492e410c764204409150989c9fda75a in 142 milliseconds
	Johnst.  emitted ae5472ffd492e410c764204409150989c9fda75a in 434 milliseconds

Firefox 19.0.2:

	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted ced3abd7a93caf2a70edb3a9a795ffdaf43f67ed in 4 milliseconds
	Johnst.  emitted ced3abd7a93caf2a70edb3a9a795ffdaf43f67ed in 3 milliseconds
	Benchmarking 1048576 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted be0d6856f1bb92a7c9ffcf770adb2c463cc75053 in 57 milliseconds
	Johnst.  emitted be0d6856f1bb92a7c9ffcf770adb2c463cc75053 in 56 milliseconds
	Benchmarking 4194304 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted 308a4f69f3aa2723e3839833c93b8ed2cb7cd6d1 in 221 milliseconds
	Johnst.  emitted 308a4f69f3aa2723e3839833c93b8ed2cb7cd6d1 in 218 milliseconds
	Benchmarking 8388608 bytes ...
	Native   emitted unavailable in 0 milliseconds
	Rusha    emitted f46d1431b9e0c5aa30424f67267445fbb2cc619c in 406 milliseconds
	Johnst.  emitted f46d1431b9e0c5aa30424f67267445fbb2cc619c in 433 milliseconds