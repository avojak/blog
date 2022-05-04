---
title: "SQLite Libretro DB"
description: "The Libretro database converted to a single SQLite file"
author: avojak
image: https://i.imgur.com/9Lf3Hw0.jpg
tags:
  - software
  - evergreen
---

The side-project that I've been focusing on lately is [Replay](https://github.com/avojak/replay): a multi-system emulator backed by [Libretro](https://libretro.com) cores.

{% include github-card.html
  user="avojak"
  repository="replay"
%}

One feature that's very important to me is the ability to automatically display information about a game when
it's added to the library. For example: when the game was released, the genre, other simmilar titles, etc. However, I don't want to need to make
network requests to a server for every game.

I came across the [database repository for Libretro](https://github.com/libretro/libretro-database) (which backs the extremely popular Retroarch application) and thought I might be able to leverage it! 

{% include github-card.html
  user="libretro"
  repository="libretro-database"
%}

## Digging Into the Data

Unfortunately the data files are stored as `.rdb` files in the repository, and the documented usage requires compiling
a `libretrodb_tool` command-line utility for viewing and querying the data. Fortunately for me, using the tool to list the content of an `.rdb` file results in
a bunch of JSON objects for each known ROM file. This means that I could easily do some processing of the data to massage it into something more useful.

## Data Processing

The first step was to run the `libretrodb_tool` against each file. From the name of the `.rdb` file we can determine the platform (e.g. Game Boy), and the manufacturer (e.g. Nintendo) (inferred by looking for a " - " separator in the filename, if present).

After grabbing the output, it can be parsed line-by-line to grab the information about each game. At this point I realized that there's a fair amount of duplicate data. I made the decision to use the ROM MD5 as the unique identifier for each game. The rationale here is that if two entries have the exact same ROM file, then they're the same game.
When I encountered duplicates, I attempted to merge the data by picking the defaulting to all values of the first result, but copying over new data to replace any null values. 

For attributes that can be common across games, I saved the values off in maps so that they could be stored in separate database tables (e.g. genres, regions, manufacturers, etc.).

## Database Schema

Once all rows had been parsed, the data is dumped into a database in separate tables. Full details of the schema can be found in the README for the project:

{% include github-card.html
  user="avojak"
  repository="libretrodb-sqlite"
%}

## Querying Data

The primary use-case for Replay is to lookup games by the MD5 of a ROM file in the library. This can easily be accomplished against the new SQLite database:

```sql
SELECT games.serial_id,
	games.release_year,
	games.release_month,
	games.display_name,
	developers.name as developer_name,
	franchises.name as franchise_name,
	regions.name as region_name,
	genres.name as genre_name,
	roms.name as rom_name,
	roms.md5 as rom_md5,
	platforms.name as platform_name,
	manufacturers.name as manufacturer_name
FROM games
	LEFT JOIN developers ON games.developer_id = developers.id
	LEFT JOIN franchises ON games.franchise_id = franchises.id
	LEFT JOIN genres ON games.genre_id = genres.id
	LEFT JOIN platforms ON games.platform_id = platforms.id
		LEFT JOIN manufacturers ON platforms.manufacturer_id = manufacturers.id
	LEFT JOIN regions ON games.region_id = regions.id
	INNER JOIN roms ON games.rom_id = roms.id
WHERE roms.md5 = "27F322F5CD535297AB21BC4A41CBFC12";
```

The total size of the SQLite file comes in at ~41MB, which is large, but not so large that it can't be easily bundled with the application.
This will be much easier than making network requests to a server for game metadata!