# vAccel plugin template

A template repo to serve as a basis for building a vAccel plugin.

# vAccel plugin API

vAccel plugins provide the glue code between vAccel User API operations and
their hardware implementations.

A vAccel plugin is a shared object, built in any programming language that can
output a shared object. This page describes the needed operations to be
implemented for a shared object to be linked as a vAccel plugin.


#### Plugin API

This repo hosts a template  for developing a vAccel plugin in `C`.

The structure of a vAccel plugin is the following:

- An `init()` function, called upon plugin initialization

```C
static int init(void) {

}
```

- A `fini()` function, called before unloading the plugin
```C
static int fini(void) {

}
```

- A definition of the `VACCEL_PLUGIN` with:

    - `.name` : The name of the plugin
    - `.version` : The version of the plugin
    - `.vaccel_version` : the vAccel library version that the plugin was built against. 
    - `.init` : the function to call upon plugin initialization (eg. `init()`)
    - `.fini` : the function to call before unloading the plugin (eg. on program exit, `fini()`)

```C
VACCEL_PLUGIN(
        .name = "vAccel template plugin",
        .version = "0.9",
	.vaccel_version = VACCEL_VERSION,
        .init = init,
        .fini = fini
)
```

At initialization, the plugin needs to register its operations to the relevant
vAccel User API operation. To do that, we use an array of `struct vaccel_op`s,
to map each function implementation to the respective API operation.  For
instance, the array could look like the following:

```C
static struct vaccel_op ops[] = {
	VACCEL_OP_INIT(ops[0], VACCEL_OP_NOOP, my_noop_function),
	VACCEL_OP_INIT(ops[1], VACCEL_OP_MINMAX, my_minmax_function),
};
```
where

```C
struct vaccel_op {
	/* operation type */
	vaccel_op_type_t type;

	/* function implementing the operation */
	void *func;

	[...]
};
```

#### vAccel main library
Before building a vAccel plugin, we should install the main vAccel library. Instructions on how to build vAccel can be found [here](https://docs.vaccel.org/quickstart/).

#### building a vAccel plugin

Now we can build a simple vAccel plugin that implements the `NOOP` user API
operation with our own custom function.

First let's clone the repo:

```bash
git clone https://github.com/nubificus/vaccelrt-plugin-template
```

We also need to install the following packages:

```bash
sudo apt-get install build-essential ninja-build pkg-config python3-pip 
sudo pip install meson
```

And we use `meson` to prepare the `build` directory:

```bash
meson setup build
```

Let's look into `./src/vaccel.c`:

```C {.line-numbers}
#include <stdio.h>

#include <vaccel.h> /* needed for vAccel specific structures (eg Session */

/* A function that will be mapped to a vAccel User API operation using
 * register_plugin_functions() */
static int my_noop_function(struct vaccel_session *sess)
{
        fprintf(stderr, "[my noop function] session: %" PRId64 "\n", sess->id);
        return VACCEL_OK;
}

/* An array of the operations to be mapped */
struct vaccel_op ops[] = {
        VACCEL_OP_INIT(ops[0], VACCEL_NO_OP, my_noop_function),
};

/* The init() function, called upon plugin initialization */
static int init(void)
{
        /* This is where the static function above `my_noop_function()`
         * gets mapped to the relevant vAccel User API operation. */
        return vaccel_plugin_register_ops(ops, sizeof(ops) / sizeof(ops[0]));
}

/* The fini() function, called before unloading the plugin */
static int fini(void)
{
        return VACCEL_OK;
}

VACCEL_PLUGIN(
        .name = "vAccel template plugin",
        .version = "0.9",
	.vaccel_version = VACCEL_VERSION,
        .init = init,
        .fini = fini
)
```

The plugin registers `my_noop_function()` to serve as the implementation of the
`VACCEL_OP_NOOP` API operation. To build it, use:

```
meson compile -C build
```

The plugin should have been created into `./build/src/`.

The output is a shared object (`libvaccel-template.so`). 

To be used as a plugin, you need to specify it in the environment variable
`VACCEL_PLUGINS` when running your vAccel application. 

See [Running a vAccel
application](https://docs.vaccel.org/build_run_app/#running-a-vaccel-application)
for more info.

