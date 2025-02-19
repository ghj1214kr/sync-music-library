# sync-music-db-bs3

keep music metadata in sync with a [sqlite](https://sqlite.org/index.html)
database. watches a directory for changes, and updates a sqlite table with
relevant id3 (or [other metadata formats](
https://github.com/borewit/music-metadata#support-for-audio-file-types)).

this module is intended to be used with media players, but is appropriate for
anything that relies on a music library.

## Original repository

<https://github.com/misterhat/sync-music-db>

## Install

    yarn add sync-music-db-bs3 better-sqlite3

or

    npm install sync-music-db-bs3 better-sqlite3

[better-sqlite3](https://www.npmjs.com/package/better-sqlite3) is a
[peerDependency](https://docs.npmjs.com/files/package.json#peerdependencies).
this module doesn't explicitly `require` it, but it takes a better-sqlite3 `db`instance in its constructor.

## Example

```typescript
const SyncMusicDb = require('../');
const Database = require('better-sqlite3');

(async () => {
    const database = new Database("example.db");
    const syncMusicDb = new SyncMusicDb({ db: database, dirs: ['D:/test/music'] });

    await syncMusicDb.createTable();

    console.time('sync');

    syncMusicDb
        .on('ready', () => console.timeEnd('sync'))
        .on('add', track => console.log(`${track.title} added`))
        .on("update", (track) => console.log(`${track.path} updated`))
        .on('remove', path => console.log(`${path} removed`))
        .on('error', err => console.error(err))
        .refresh();
})();
```

## API

### SyncMusicDb.TRACK\_ATTRS

the columns in the `library` table.

    [
        'path', 'mtime', 'title', 'artist', 'album', 'year', 'duration', 'track_no',
        'tags', 'is_vbr', 'bitrate', 'codec', 'container'
    ]

### syncMusicDb = new SyncMusicDb({ db, dirs, delay = 1000 })

create an `EventEmitter` to sync the specified `dirs` directory array to a
[better-sqlite3](https://www.npmjs.com/package/better-sqlite3) `db` instance.

`delay` specifies how long to wait for file changes (in ms) before reading them.

### async syncMusicDb.createTable()

create the `library` table in the `sqliteDb` instance.

### syncMusicDb.addDirs(dirs)

add `dirs` to `syncMusicDb`.

`.refresh` call is required to take effect.

### syncMusicDb.removeDirs(dirs)

remove `dirs` from `syncMusicDb`

`.refresh` call is required to take effect.

### syncMusicDb.refresh()

do an initial sync with the specified `dirs` and begin watching it for
new changes.

### async syncMusicDb.close()

stop syncing and watching `dirs`.

### syncMusicDb.on('synced', isSynced => {})

is `sqliteDb` up-to-date with `dirs`?

### syncMusicDb.on('ready', () => {})

the initial sync has finished (from a `.refresh` call).

if `dirs` is empty, this event will not triggered.

### syncMusicDb.on('add', track => {})

`track` has been added.

### syncMusicDb.on('update', track => {})

`track` has been updated.

### syncMusicDb.on('remove', path => {})

`path` has been removed.

### syncMusicDb.isReady

is `syncMusicDb` listening to live `dirs` changes (after initial scan)?

### syncMusicDb.isSynced

is all the metadata from `dirs` stored in `db`?

## License

LGPL-3.0+
