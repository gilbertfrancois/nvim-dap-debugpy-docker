# Debug Python inside Docker container using NVim DAP and debugpy
Gilbert François Duivesteijn <info@gilbertfrancois.com>

## Abstract

This is a guide on how to setup a development environment for python in a docker container using NeoVim as the editor and the `remote` plugin to connect to the container.

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
            name = 'Attach to Docker python debugpy',
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

__Note:__ The `pathMappings` section is important. It tells the debugger where to
find the source code in the container. In this case, the source code is in the
`/app` directory in the container. If you miss this step, the breakpoints will
not work.

For a full example, see my personal nvim config repository: [gilbertfrancois/nvim](https://github.com/gilbertfrancois/nvim)

## Building the docker image locally

To build the docker image locally, you can use the following command:

```bash
docker buildx build -t my-python-container:latest --local .
```

## Starting the docker container

To start the docker container, you can use the following command:

```bash
docker run --rm -v $(pwd):/app -p 5678:5678 my-python-container:latest
```

__Note:__ The `-v $(pwd):/app` flag mounts the current directory to the `/app`. This
is important because the `pathMappings` section in the `debug.lua` file expects
the source code to be in the `/app` directory.

## Start debugging in NeoVim

To start debugging in NeoVim, you can use the following command:

```vim
:DapContinue
```

