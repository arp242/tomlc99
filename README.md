# tomlc99

TOML in c99; v1.0 compliant.

If you are looking for a C++ library, you might try this wrapper: [https://github.com/cktan/tomlcpp](https://github.com/cktan/tomlcpp).

## Usage

Please see the `toml.h` file for details. What follows is a simple example that
parses this config file:

```toml
[server]
	host = "www.example.com"
	port = 80
```

The steps for getting values from our file is usually :

1. Parse the whole TOML file.
2. Get a single table from the file.
3. Find a value from the table.
4. Then, free up that memory if needed.

Below is an example of parsing the values from the example table.

1. Parse the whole TOML file.

```c
FILE* fp;
toml_table_t* conf;
char errbuf[200];

/* Open the file and parse content */
if (0 == (fp = fopen("path/to/file.toml", "r"))) {
	return handle_error();
}
conf = toml_parse_file(fp, errbuf, sizeof(errbuf));
fclose(fp);      
if (0 == conf) {
	return handle_error();
}


/* Alternatively, use `toml_parse` which takes a string rather than a file. */
conf = toml_parse("A null terminated string that is TOML\0", errbuf, sizeof(errbuf));
```

2. Get a single table from the file.

```c
toml_table_t* server;

/* Locate the [server] table. */
if (0 == (server = toml_table_in(conf, "server"))) {
	return handle_error();
}
```

3. Find a value from the table.

```c
/* Extract 'host' config value. */
toml_access_t host = toml_string_in(server, "host");
if (!host.ok) {
	toml_free(conf);
	return handle_error();
}

toml_access_t port = toml_int_in(server, "port");
if (!port.ok) {
	toml_free(conf);
	free(host.u.s);
	return handle_error();
}

printf("host %s\n", host.u.s);
printf("port %d\n", port.u.i);

```

4. Then, free up that memory if needed.

```c
/* Use `toml_free` on the table returned from `toml_parse[_file]`. */
toml_free(conf);

/* Free any string values returned from access functions. */
free(host.u.s);
```

### Accessing Table Content

Tables are dictionaries where lookups are done using string keys. In
general, all access methods on tables are named `toml_*_in(...)`.

Keys in tables can be iterrogated using a key index:

```c
toml_table_t* tab = toml_parse_file(...);
for (int i = 0; ; i++) {
    const char* key = toml_key_in(tab, i);
    if (!key) break;
    printf("key %d: %s\n", i, key);
}
```

Once you know a key and its content type, you can obtain its content in the table by one of these methods:
```c
toml_string_in(tab, key);
toml_bool_in(tab, key);
toml_int_in(tab, key);
toml_double_in(tab, key);
toml_timestamp_in(tab, key);
```


## Building

A normal *make* suffices. Alternately, you can also simply include the
`toml.c` and `toml.h` files in your project.

## Testing

To test against the standard test set provided by BurntSushi/toml-test:

```sh
% make
% cd test1
% bash build.sh   # do this once
% bash run.sh     # this will run the test suite
```


To test against the standard test set provided by iarna/toml:

```sh
% make
% cd test2
% bash build.sh   # do this once
% bash run.sh     # this will run the test suite
```
