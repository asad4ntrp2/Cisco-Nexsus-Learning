# Nexus Learning - Version Archive

This directory contains snapshots of key project files from each tagged version
of the Cisco Nexus Student Learning Guide.

## Folder Structure

```
versions/
├── README.md          ← This file
├── v1.1.0/            ← Initial GitHub release
│   ├── index.html
│   ├── README.md
│   └── VERSION.md
├── v1.2.0/            ← Inline SVG diagrams + landing page
│   ├── index.html
│   ├── README.md
│   └── VERSION.md
├── v1.3.0/            ← Vibrant gradient diagrams, branding watermarks
│   ├── index.html
│   ├── README.md
│   └── VERSION.md
├── v1.3.1/            ← Documentation & version history updates
│   ├── index.html
│   ├── README.md
│   └── VERSION.md
├── v2.0.0/            ← DAD, EVPN Guide, Stacking vs VPC chapters
│   ├── index.html
│   ├── README.md
│   ├── VERSION.md
│   └── continue.md
└── v2.1.0/            ← Private VLANs (PVLANs) chapter
    ├── index.html
    ├── README.md
    ├── VERSION.md
    └── continue.md
```

## Version History

| Version | Date       | Commit    | Description |
|---------|------------|-----------|-------------|
| v1.1.0  | 2026-02-14 | `f525476` | Initial GitHub release of the Cisco Nexus Student Learning Guide |
| v1.2.0  | 2026-02-14 | `9325676` | Replace Draw.io viewer with inline SVG diagrams; add landing page |
| v1.3.0  | 2026-02-14 | `de6f93e` | Vibrant gradient diagrams, branding watermarks, and diagrams gallery |
| v1.3.1  | 2026-02-14 | `b118da7` | Update documentation, version history, and landing page |
| v2.0.0  | 2026-02-18 | `08bf94c` | Add DAD, EVPN Comprehensive Guide, Stacking vs VPC chapters |
| v2.1.0  | 2026-02-18 | `e4e7f68` | Add Private VLANs (PVLANs) chapter with diagram |

## How These Files Were Extracted

Each version folder was created by extracting files from the corresponding Git
commit using:

```bash
git show <commit>:<file> > versions/vX.Y.Z/<file>
```

Files extracted per version (when they exist at that commit):
- **index.html** - The main single-page HTML learning guide
- **README.md** - Project documentation at that point in time
- **VERSION.md** - Detailed version/changelog information
- **continue.md** - Development continuation notes (added in v2.0.0)

## Notes

- The `v1.2.0` folder uses commit `9325676` (the later of two v1.2.0 commits),
  which replaced the Draw.io viewer with inline SVG diagrams.
- The `continue.md` file first appears in v2.0.0 and is not present in earlier
  versions.
- These are snapshots of individual files only; assets in `diagrams/` and `docs/`
  subdirectories are not included here. Use `git checkout <commit>` to get a
  complete working tree for any version.
