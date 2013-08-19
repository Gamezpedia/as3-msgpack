# as3-msgpack v1.0.1
<p>as3-msgpack is a implementation of MessagePack specification for Actionscript3 language (Flash, Flex and AIR).</p>
<p><strong>Download the lastest tag:</strong> https://github.com/loteixeira/as3-msgpack/archive/v1.0.1.zip<br>
<strong>See online documentation:</strong> http://disturbedcoder.com/files/as3-msgpack/</p>

## about message pack format
<p>MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON but it's faster and smaller. For example, small integers (like flags or error code) are encoded into a single byte, and typical short strings only require an extra byte in addition to the strings themselves.</p>
<p>Check the website: http://msgpack.org</p>

## supported types
<p>Message pack specification was built to work mostly with primitive types. You shouldn't expect a serialization library, because msgpack is similiar to JSON - but is based on binary data instead of text.<br>
The types available are: signed/unsigned integer, single/double precision floating point, nil (null), boolean, array, associative array (map) and raw buffer.<br>
These types are mapped to Actionscript3 as the following:</p>
* signed integer -> int
* unsigned integer -> uint
* single/double precision floating point -> Number
* nil -> null
* boolean -> Boolean
* array -> Array
* associative array -> Object
* raw buffer -> String or ByteArray

## changes
### 1.0.0 to 1.0.1
* Using make tool to compile the binaries instead of batch files.
* Improving README.md
* Creating msgpack.org.md (as3-msgpack API document for msgpack.org)
### 0.4.1 to 1.0.0
* The access interface has been totally modified. Now isn't possible to use singletons, to read/write message pack data you'll need create an object.
* The names of classes and files were updated. The class MessagePack became MsgPack and the library MessagePack.swc became msgpack.swc.
* Stream reading is available. If you read data from a buffer (IDataInput) and the object is not complete, the library stores the current status and then wait for more bytes.

## folders
* bin: binaries (library and test applications);
* lib: libraries used by test applications;
* doc: documentation generated by asdoc;
* src_lib: source code of the library;
* src_test: source code of test applications (including a server and a client which exchange streaming messagepack packets through sockets);

## examples
### basic usage
<p>The usage of MsgPack class is very simple. You need create an object and call read and write methods.</p>
```actionscript
// message pack object created
var msgpack:MsgPack = new MsgPack();

// encode an array
var bytes:ByteArray = msgpack.write([1, 2, 3, 4, 5]);

// rewind the buffer
bytes.position = 0;

// print the decoded object
trace(msgpack.read(bytes));
```

### streaming
<p>You may read streaming data making successive calls to msgpack.read method. Each MsgPack object can handle one stream at time.<br>
If all bytes are not available, the method returns incomplete (a special object).</p>
```actionscript
package
{
	import flash.events.*;
	import flash.net.*;

	import org.msgpack.*;

	public class Streamer
	{
		private var msgpack:MsgPack;
		private var socket:Socket;

		public function Streamer()
		{
			msgpack = new MsgPack();

			// flash sockets implements the interfaces IDataInput and IDataOutput.
			// thus, these objects may be directly connected to as3-msgpack.
			socket = new Socket();
			socket.addEventListener(ProgressEvent.SOCKET_DATA, socketData);
			// connecting to our hypothetical server
			socket.connect("localhost", 5555);
		}

		public function send(data:*):void
		{
			// we'll write the encoded object straight into the socket (since it implements IDataOutput interface).
			msgpack.write(data, socket);
			socket.flush();
		}

		private function socketData(e:ProgressEvent):void
		{
			// until the data is ready, this call returns incomplete.
			var data:* = msgpack.read(socket);

			// is the object complete?
			if (data != incomplete)
			{
				trace("OBJECT READY:");
				trace(data);
			}
		}
	}
}
```

### using flags
<p>Currently there are two flags which you may use to initialize a MsgPack object:</p>
* MsgPackFlags.READ_RAW_AS_BYTE_ARRAY: message pack raw data is read as byte array instead of string;
* MsgPackFlags.ACCEPT_LITTLE_ENDIAN: MsgPack objects will work with little endian buffers (message pack specification defines big endian as default).

```actionscript
var msg:MsgPack;

// use logical operator OR to set the flags.
msgpack = new MsgPack(MsgPackFlags.READ_RAW_AS_BYTE_ARRAY | MsgPackFlags.ACCEPT_LITTLE_ENDIAN);
```
