# Debug Python inside Docker container using NVim DAP and debugpy

_Gilbert François Duivesteijn <info@gilbertfrancois.com>_

## Abstract

This is a guide on how to setup a development environment for python in a
docker container using NeoVim as the editor and the `remote` plugin to connect
to the container. Although this is just showing how to setup python, you can
use the same setup for other languages as well, e.g. C and C++ using `codelldb`.

[![Demo video](https://img.youtube.com/vi/uyMtCpUW-xk/0.jpg)](https://www.youtube.com/watch?v=uyMtCpUW-xk)

## Setup in NeoVim

In your debug.lua section, you can add the following:

```lua
    -- Python debugging setup
    local debugpy_path = require('mason-registry').get_package('debugpy'):get_install_path()
    require('dap-python').setup(debugpy_path .. '/venv/bin/python')
    dap.adapters.python = {
        type = 'server',
        host = 'localhost',
        port = 5678,
        options = {
            source_filetype = 'python',
        },
    }
    dap.configurations.python = {
        {
            type = 'python',
            request = 'attach',
            connect = {
                host = 'localhost',
                port = 5678,
            },
            mode = 'remote',
            name = 'Attach to Docker python in /app',
            redirectOutput = true,
            justMyCode = true,
            pathMappings = {
                {
                    localRoot = vim.fn.getcwd(),
                    remoteRoot = '/app',
                },
            },
        },
    }
```

**Notes:** 
- The `pathMappings` section is important. It tells the debugger where to
find the source code in the container. In this case, the source code is in the
`/app` directory in the container. If you miss this step, the breakpoints will
not work.
- You can add more configurations for different python projects. Just copy the
`dap.configurations.python` section and change the `name` and `pathMappings` to
fit your project.
- Enable `redirectOutput` to see the output of the python script in the terminal
window.


For a full example, see my personal nvim config repository:
[gilbertfrancois/nvim](https://github.com/gilbertfrancois/nvim)

## Building the docker image locally

To build the docker image locally, you can use the following command:

```bash
docker buildx build -t my-python-container:latest --local .
```

## Starting the docker container

The CMD inside the docker container is set to (see the Dockerfile):

```bash
python -Xfrozen_modules=off -m debugpy --listen 0.0.0.0:5678 --wait-for-client /app/main.py
```

**Notes:** 
- The `--wait-for-client` flag is important. It tells the debugpy server to wait
for a client to connect before starting the python script.
- The `-Xfrozen_modules=off` flag is needed to disable the frozen modules feature
in python. This is needed because the debugpy server needs to be able to import
the `debugpy` module.

This starts the debugpy server and waits for a client to connect. Start the container with:

```bash
docker run --rm -v $(pwd):/app -p 5678:5678 my-python-container:latest
```

**Note:** The `-v $(pwd):/app` flag mounts the current directory to the `/app`. This
is important because the `pathMappings` section in the `debug.lua` file expects
the source code to be in the `/app` directory.

## Start debugging in NeoVim

To start debugging in NeoVim, you can use the following command:

```vim
:DapContinue
```

This will connect to the debugpy server running in the docker container at 0.0.0.0:5678. The breakpoints should work now
and you can start debugging your python code.
