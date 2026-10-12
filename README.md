# Lucee Extension SMB Resource Provider

[![Java CI](https://github.com/lucee/extension-smb/actions/workflows/main.yml/badge.svg)](https://github.com/lucee/extension-smb/actions/workflows/main.yml)

This extension provides SMB (Server Message Block) / CIFS (Common Internet File System) virtual file system support for Lucee, allowing you to read, write, and manipulate files on Windows network shares and Samba servers using standard CFML file functions.

## Documentation

- [Lucee Virtual File Systems](https://docs.lucee.org/guides/lucee-5/virtual-file-systems.html)
- [Working with Files and Directories](https://docs.lucee.org/guides/Various/working-with-files.html)

## Usage

Access SMB shares using the `smb://` scheme with standard CFML file functions:

```cfml
// Basic usage with credentials in URL
smbPath = "smb://username:password@server/share/path/";

// Check if directory exists
if ( directoryExists( smbPath ) ) {
    // List files
    files = directoryList( smbPath );

    // Read a file
    content = fileRead( smbPath & "document.txt" );

    // Write a file
    fileWrite( smbPath & "output.txt", "Hello SMB!" );
}

// With domain authentication
smbPath = "smb://DOMAIN;username:password@server/share/";
```

### URL Format

```text
smb://[domain;]username:password@host[:port]/share/path/
```

- **domain** - Optional Windows domain
- **username** - SMB username
- **password** - SMB password
- **host** - Server hostname or IP
- **port** - Optional port (default: 445)
- **share** - Share name
- **path** - Path within the share

### Supported Operations

- `directoryCreate()` / `directoryDelete()` / `directoryExists()` / `directoryList()`
- `fileRead()` / `fileWrite()` / `fileDelete()` / `fileExists()`
- `fileCopy()` / `fileMove()`
- `getFileInfo()` / `fileSetLastModified()`

## Configuration

The extension works out of the box with sensible defaults. To customise the SMB settings, add arguments to your `.CFConfig.json`:

```json
{
  "resourceProviders": [
    {
      "scheme": "smb",
      "class": "org.lucee.extension.resource.smb.SMBResourceProvider",
      "arguments": {
        "minVersion": "SMB202",
        "signingPreferred": "true"
      }
    }
  ]
}
```

### Available Options

| Option | Default | Description |
|--------|---------|-------------|
| `minVersion` | `SMB1` | Minimum SMB protocol version: `SMB1`, `SMB202`, `SMB210`, `SMB300`, `SMB302`, `SMB311` |
| `maxVersion` | `SMB311` | Maximum SMB protocol version |
| `soTimeout` | `35000` | Socket timeout in milliseconds |
| `connTimeout` | `35000` | Connection timeout in milliseconds |
| `responseTimeout` | `30000` | SMB response timeout in milliseconds |
| `signingPreferred` | `false` | Prefer SMB signing when available |
| `signingEnforced` | `false` | Require SMB signing (fails if server doesn't support it) |
| `encryptionEnabled` | `false` | Enable SMB3 encryption |
| `dfsDisabled` | `true` | Disable DFS (Distributed File System) lookups |
| `resolveOrder` | `DNS` | Name resolution order (e.g., `DNS`, `LMHOSTS,DNS,WINS,BCAST`) |

### Security Recommendations

For modern environments, consider:

```json
{
  "arguments": {
    "minVersion": "SMB300",
    "signingEnforced": "true",
    "encryptionEnabled": "true"
  }
}
```

This enforces SMB3+ with signing and encryption, which is the most secure configuration.

## Requirements

- Lucee 7.x or later
- Java 17 or later

## Technical Details

This extension uses [jcifs-codelibs 3.0.0](https://github.com/codelibs/jcifs), a modern fork of the JCIFS library with SMB2/SMB3 support.

## Issues

[Lucee JIRA - SMB Issues](https://luceeserver.atlassian.net/issues/?jql=labels%20%3D%20smb)

## Versioning and releases

Works like Lucee core (LDEV-6516): every code change gets its own version number. The version is not edited by hand.
`pom.xml` uses a CI-friendly `${revision}` built from `extension.version.base` (`1.0.0`), a build number and
`extension.version.qualifier` (`-SNAPSHOT`).

* **Every push to `main`** (a merged PR or a direct commit) takes the highest existing build of the base, from the git tags,
  Maven Central and the Central snapshot repository, adds 1, builds, tests, publishes that version (e.g. `1.0.0.12-SNAPSHOT`)
  to the [Central snapshot repository](https://central.sonatype.com/repository/maven-snapshots/) and creates the tag `1.0.0.12`.
  No `[release]` keyword and no "set version" commit is needed.
* Pushes run one after another (workflow concurrency queue), so two quick merges get two different numbers.
* PR builds, forks and runs without the deploy secrets build and test `1.0.0.0-SNAPSHOT` and never use up a number.
  A commit with `[skip ci]` in its message does not start a build, so it does not get a number either.
  Like Lucee core there is no exception for documentation-only changes: they get a new snapshot too.
* **Release candidate or final release**, either way gets the next number:
  * like Lucee core, commit a change of `extension.version.qualifier` in `pom.xml` to `-RC` (release candidate) or to empty
    (final release). That push publishes e.g. `1.0.0.13-RC` to Maven Central. Afterwards commit `-SNAPSHOT` back
    ("new cycle"), otherwise every following push publishes another RC.
  * or run the workflow by hand on `main` (Actions > Run workflow), tick `deploy` and pick `RC`, `BETA`, `ALPHA` or `release`
    as qualifier. That publishes the current code once, without a commit; the pom keeps `-SNAPSHOT`.
* Central keeps snapshots for a limited time only; the git tags keep the numbers unique after a snapshot was removed.
* To start a new line change `extension.version.base`.
