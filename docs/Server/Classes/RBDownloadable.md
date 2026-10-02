### Class
`RustBuster2016Server.RBDownloadable`

### Description
Describes a file that RustBuster should push to connecting clients, registered via
[`API.AddFileToDownload`](API.md#static-bool-addfiletodownloadrbdownloadable-rbd). This is the
server-side half of the download pipeline that
[`Facepunch.Load.Loader`](../../Client/Classes/CustomMap.md) / RustBuster's own downloadable queue
consumes on the client - useful for shipping custom asset bundles (textures, prefabs, custom maps, see
[`CustomMap.md`](../../Client/Classes/CustomMap.md) on the client side) or any other file your plugin
needs present on every client.

There are two ways to source the file: directly from a path on the server's disk (compressed with LZ4 and
embedded in the handshake), or from a direct URL download (the client fetches it itself via HTTP/HTTPS).

### Constructors

#### `RBDownloadable(string clientpath, string filepath, bool protectfromoverwrite = false)`
Reads the file at `filepath` (on the **server's** disk) immediately, LZ4-compresses it (unless it ends in
`.unity3d`, which is sent uncompressed), and computes its SHA1 checksum - all up front, in the
constructor. Logs an error and leaves the object unusable if `filepath` doesn't exist.
- `clientpath` - the folder the file will be downloaded into on the client, e.g. `"\\MyDirectory\\MySubDir"`.
- `filepath` - the path to the file on the server.
- `protectfromoverwrite` - if `true`, the client will not overwrite its local copy even if it differs
  from the server's (useful for files players are allowed to customize locally).

#### `RBDownloadable(string filename, string clientpath, string url, string sha1offile, bool protectfromoverwrite = false)`
Describes a file the client should download directly from a URL instead of through RustBuster's own
handshake - better for large files (e.g. custom map bundles).
- `filename` - the file's name, e.g. `"hapis_island.unity3d"`.
- `clientpath` - the folder the file will be downloaded into on the client.
- `url` - the direct download link.
- `sha1offile` - the SHA1 hash of the file at `url`, used by the client to verify/skip re-downloading.
- `protectfromoverwrite` - same as above.

### Members
- `string Clientpath { get; }`, `string Filepath { get; }` (empty for URL downloads),
  `string Filename { get; }`.
- `byte[] Filebytes { get; }` (`null` for URL downloads), `byte[] Compressedbytes { get; }`,
  `string Base64String { get; }` (base64 of the compressed bytes).
- `string SHA1Checksum { get; }`.
- `bool IsURLDownload { get; }`, `string URL { get; }` (only set for URL downloads).
- `bool ProtectFromOverwrite { get; }`.

### Example

Local file, pushed through RustBuster's own handshake:
```csharp
public override void Initialize()
{
    string path = Path.Combine(ModuleFolder, "mytexture.unity3d");
    RBDownloadable file = new RBDownloadable("\\MyFirstPlugin", path);
    API.AddFileToDownload(file);
}
```

Large file, downloaded directly from a URL (e.g. a custom map - see
[`CustomMap.md`](../../Client/Classes/CustomMap.md) for the client-side half):
```csharp
public override void Initialize()
{
    RBDownloadable map = new RBDownloadable(
        "hapis_island.unity3d",
        "\\Maps",
        "https://example.com/downloads/hapis_island.unity3d",
        "0123456789ABCDEF0123456789ABCDEF01234567");

    API.AddFileToDownload(map);
}

public override void DeInitialize()
{
    API.DeleteFileFromDownloadByName("hapis_island.unity3d");
}
```
