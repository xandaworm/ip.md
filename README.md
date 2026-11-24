# IP.MD

**Network diagrams as code in Markdown**

IP.MD turns a set of Markdown files into a network diagram.
(Early Development)

## How it works

```
network/
|――rack/
|   |――router.md
|   |――switch.md
|   |――vm-server.md
|   |――vm-server/
|       |――vm.md
```

Each `.md` file describes a singular device and its information (name, type, IP, connections, and services). IP.MD reads the folder and lays it out with [ELK](https://github.com/kieler/elkjs).

## Planned features

- Rendering to SVG
- Live previewing
- Service mapping
- Extension support

## License

[MIT](LICENSE).