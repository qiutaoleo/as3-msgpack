# as3-msgpack v1.0.0
<p>as3-msgpack is a implementation of MessagePack specification for Actionscript3 language (Flash, Flex and AIR).</p>
<p>Download the lastest tag: not available yet! (you may clone the repo)</p>
<p>See online documentation: http://disturbedcoder.com/files/as3-msgpack/</p>

## about message pack format
<p>MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON but it's faster and smaller. For example, small integers (like flags or error code) are encoded into a single byte, and typical short strings only require an extra byte in addition to the strings themselves.</p>
<p>Check the website: http://msgpack.org</p>

## supported types
<p>Message pack was built to work with primitive types (similiar to C). You shouldn't expect a serialization library, because msgpack is very similiar to JSON - but the data is binary, not text.</p>
<p>The available types are: signed/unsigned integer, single/double precision floating point, nil (null), boolean, array, associative array (map) and raw buffer.</p>
<p>These types are mapped to Actionscript3 like the following:</p>
* signed integer -> int
* unsigned integer -> uint
* single/double precision floating point -> Number
* nil -> null
* boolean -> Boolean
* array -> Array
* associative array -> Object
* raw buffer -> String or ByteArray

## changes
* The access interface has been totally modified. Now isn't possible to use singletons, everytime you need to read/write message pack data you'll need to create an instance of the object.
* Classes and files names were modified. The class MessagePack became MsgPack and the library MessagePack.swc became msgpack.swc.
* Stream reading is possible now. If you read data from a buffer (IDataInput) and the object is not complete, the library stores the current status and then wait for more bytes.

## folders
* bin: binaries (library and test applications);
* doc: documentation generated by asdoc;
* lib: libraries used by test applications;
* src_lib: source code of the library;
* src_test: source code of the test applications and
* tools: batch files for compilation.

## examples
### basic usage
<p>Using MsgPack class is very simple. You need create an object and basically use the read and write methods.</p>
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
<p>You may read streaming data making successive calls to msgpack.read method. Each MsgPack object can handle one stream at time.</p>
<p>If all bytes are not available, the method returns incomplete (a special object).</p>
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
			// we'll write the encoded object straight into socket (since it implements IDataOutput interface).
			msgpack.write(data, socket);
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
<p>Currently there are two flags which you may use to initialize the MsgPack object:</p>
* Factory.READ_RAW_AS_BYTE_ARRAY: message pack raw data is read as byte array instead of string;
* Factory.ACCEPT_LITTLE_ENDIAN: MsgPack objects will work with little endian buffers (message pack specification defines big endian as default).

```actionscript
var msg:MsgPack;

// use logical operator OR to set the flags.
msgpack = new MsgPack(Factory.READ_RAW_AS_BYTE_ARRAY | Factory.ACCEPT_LITTLE_ENDIAN);
```