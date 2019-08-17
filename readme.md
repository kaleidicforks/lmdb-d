
This is a D language binding for the lmdb (lightning memmapped DB) library.

Prerequisites:
- You must have liblmdb.so 0.9.21 or newer installed on your system
- A working D compiler and Dub.

Usage:
- Add 'lmdb' as a dependancy to your dub project file.
- Please also read the dub documentation for details.
- Then use 

```
import lmdb;      /* Import the D binding module into scope */
import lmdb_oo;   /* Import optionally a more OO layer, WIP */
```

Todos:
- Added real unit- and module-tests
- Create D style OO layer instead of just porting the C++ approach.

Example:

```d

void main()
{
	import std.stdio;
	import lmdb;
	import lmdb_oo;
	import std.exception;
	import core.stdc.errno;
	import std.conv : to;

	auto env = MdbEnv.create(0);
	env.open("test.lmdb",0);
	env.set_mapsize(1UL * 1024UL * 1024UL * 1024UL); // 1GB
	writeln(env);
	auto txn = MdbTxn.begin(env.handle,null,0);
	auto dbi = MdbDbi.open(txn.handle);
/*	auto key = new MDB_val("foo.bar");
	auto data = new MDB_val("bar.foo");
	enforce(dbi.put(txn.handle,key,data,0) == true);
	*/
	auto key2 = MDB_val(cast(void*) "foo".ptr,3UL);
	auto val2 = MDB_val(cast(void*)"bar".ptr,3UL);
	enforce(mdb_put(txn.handle,dbi.handle,&key2,&val2,0) == 0, "C put failed");
	txn.commit();
	writeln("success");

	auto readtxn = MdbTxn.begin(env.handle,null,MdbFlag.readOnly);
	auto cursor = MdbCursor.open(readtxn.handle,dbi.handle);
	auto key3 = new MdbVal("");
	auto val3 = new MdbVal("");
	auto keyHandle = key3.handle();
	auto valHandle = val3.handle();
	while(cursor.get(keyHandle,valHandle,MDB_cursor_op.MDB_NEXT))
	{
		auto sizeKey = key3.size();
		auto sizeVal = val3.size();
		writefln("%s: %s",key3.data!char[0..sizeKey], val3.data!char[0..sizeVal]);
	}
	cursor.close();
	readtxn.abort();
}
```

