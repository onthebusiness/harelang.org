---
title: Chronology in Hare
author: Byron Torres
date: 2022-04-17
---

Date and time support has come to Hare, with the introduction of two new
modules to the standard library, written by Hare contributors Byron
Torres (Hi, there! Today's blog guest) and Vlad-Stefan Harbuz.

- [time::chrono][c]
- [datetime][dt]

With these, we are now able to handle human notions of time: dates, the
Gregorian calendar, wall-times, timezones, the "Olson" Timezone
database, timezone transitions (daylight saving time, etc.), and
timescales (leap seconds, etc.). We can also parse, format, and
programmatically construct datetimes of high precision, and perform
elementary calendrical arithmetic.

Dates and times in software are notoriously complicated and error prone.
It's hard to get right, as many other library designers have expressed.
There are challenges within every strata of code, concerning all the
aforementioned affairs. Making a library which was robust and complete
enough, yet had a user-friendly API, was a mighty challenge.

Nonetheless, we've tried our best to design our library to be
easy-to-use, modular, performant, and above all else, correct. It will
need a few tweaks before it reaches its high aspirations, but today's
library can handle the most common problems a user will come across.

Let's go over Hare's new additions to the standard library.


## The chrono module

At the core exists the [time::chrono][c] submodule, a natural extension
of the [time][t] module. It centres around the [chrono::moment][m] type,
itself a nephew of the [time::instant][i] type. While instants are
simple scalar objects without a plane of reference, moments carry
information about their temporal context, like their locality
(timezone). Moments are an abstracted form of what you may know as a
datetime, and will largely be used by the stdlib and third-party modules
which seek cohesion with the rest of ecosystem.

```hare
// A date & time, within a locality, intepreted via a chronology
type moment = struct {
	// The ordinal day (on Earth or otherwise)
	// since the Hare epoch (zeroth day) 1970-01-01
	date: epochal,

	// The time since the start of the day
	time: time::duration,

	// The timezone used for interpreting a moment's date and time
	loc: locality,

	// The current [[zone]] this moment observes
	zone: zone,
};
```

time::chrono includes the [timezone][tz] type, best defined as "a region
(physical or abstract) where all wall-clocks agree". Provided are some
default timezones, like LOCAL (Local time), UTC, TAI, MTC, etc. A
[TZif][tzif] parser is used to read the system's installed [Olson/IANA
Timezone database][tzdb], utilised by the [chrono::tz][tzfn] function.

```hare
use time::chrono;

// A new moment equivalent to 2022-04-17 15:30 UTC
// (19098 days since 1970-01-01, 55800 seconds since 00:00)
let m = chrono::new(19098, 55800, chrono::UTC);

// same moment in time, but in a different locality.
let m = chrono::in(&chrono::tz("Pacific/Honolulu"), m);
```

On a Unix-like machine, the Timezone database is a set of files located
at `/usr/share/zoneinfo`. The [chrono::LOCAL][LOCAL] timezone is set
during initialisation, according to the system's configured
`/etc/localtime`.

```
$ ls -F /usr/share/zoneinfo
Africa/      Cuba     GMT+0        Japan              Pacific/    Turkey
America/     EET      GMT-0        Kwajalein          Poland      tzdata.zi
Antarctica/  Egypt    GMT0         leapseconds        Portugal    UCT
Arctic/      Eire     Greenwich    leap-seconds.list  posix/      Universal
Asia/        EST      Hongkong     Libya              posixrules  US/
Atlantic/    EST5EDT  HST          MET                PRC         UTC
Australia/   Etc/     Iceland      Mexico/            PST8PDT     WET
Brazil/      Europe/  Indian/      MST                right/      W-SU
Canada/      Factory  Iran         MST7MDT            ROC         zone1970.tab
CET          GB       iso3166.tab  Navajo             ROK         zone.tab
Chile/       GB-Eire  Israel       NZ                 SECURITY    Zulu
CST6CDT      GMT      Jamaica      NZ-CHAT            Singapore
$ realpath /etc/localtime
/usr/share/zoneinfo/Europe/London
```

Where Hare differs from most other languages is the feature of
*timescales*, the notion of a "dimension" on which instants exist, a
dimension of reference and measure.

The [chrono::timescale][ts] type embodies this notion. Its role allows
for careful and deliberate handling of things like leap seconds at a
stock exchange, or timekeeping in scientific contexts at an astronomic
observatory.

```hare
// Represents a scale of time; a time standard
type timescale = struct {
	name: str,
	abbr: str,
	to_tai: *ts_converter,
	from_tai: *ts_converter,
};

// Converts one [[time::instant]] in one [[chrono::timescale]] to another
type ts_converter = fn(i: time::instant) (time::instant | time::error);
```

Hare uses TAI ([International Atomic Time][taiw]) as the central
timescale, as [chrono::tai][tai]. TAI has the useful properties of being
continuous and regular.

There's also UTC ([Coordinated Universal Time][utcw]), as
[chrono::utc][utc]. Note that we're referring to *the UTC timescale*,
not the incidentally named "UTC" timezone, which could be more
unambiguously referred to as "[UTC+00:00][utc0]" or "Zulu time". UTC is
definitionally based upon TAI. The two are offsetted by a variable number
of seconds (at the time of writing, 37). Thus, UTC is not continuous and
contains leap seconds.

MTC ([Coordinated Mars Time][mtcw]) also makes a notable appearance as
[chrono::mtc][mtc]. Martian seconds run ~3% slower, as a "sol" (Martian
solar day) takes 24 hours and 39 minutes in Earth time. Hare is not yet
a space-faring language, but we can take small steps to get there[^mars].

Hare allows for the conversion of instants between timescales, and
leverages the [time::error][e] types if and when users decide to deal
with edge cases. chrono::utc in particular reads from
`/usr/share/zoneinfo/leap-seconds.list`, a plaintext list of leap
seconds, maintained by and fetched from observatories.

```hare
let utc = time::now(time::clock::REALTIME);  // 1650123000
let tai = chrono::utc.to_tai(utc)!;          // 1650123037 (+37)
let utc = chrono::utc.from_tai(tai)!;        // 1650123000
```


## The datetime module

The [datetime][dt] module concerns itself with human-oriented forms of
time. It centres around the [datetime::datetime][dtdt] type, an
extension of the chrono::moment type. A typical third-party library will
create their own type with its own units. This datetime type is designed
for the "Gregorian chronology".

```hare
type datetime = struct {
	chrono::moment,
	era: (void | int),
	year: (void | int),
	month: (void | int),
	day: (void | int),
	yearday: (void | int),
	isoweekyear: (void | int),
	isoweek: (void | int),
	week: (void | int),
	weekday: (void | int),
	hour: (void | int),
	min: (void | int),
	sec: (void | int),
	nsec: (void | int),
};
```

Hare defines a "chronology" as a system used to name and order moments
in time. In practice, a chronology is the combination of a calendar (for
handling days) and a wall clock (for handling times throughout a day).
The de facto international chronology is defined in the [ISO 8601
standard][8601], which uses the proleptic Gregorian calendar and the
24-hour timekeeping system. Alternatively, a theoretical third-party
"mayan" module could implement the [Mesoamerican Long Count
calendar][long].

Datetimes hold the same information as moments (locality, date, time),
but also hold "chronological" values corresponding to units like
"years", "hours", etc. These values are cached for performance and
utility, hence the `(void | int)` type. Datetimes created and handled by
the datetime module are guaranteed to be internally valid and
consistent, given you don't modify the fields yourself.

Here's what creating a datetime with [datetime::new][new] looks like. We
specify the timezone and zone offset first, then its fields.

```hare
// short form
// 0000-01-01 00:00:00.000000000 +0000 UTC, UTC
let dt = datetime::new(chrono::UTC, 0000);

// 1995-09-01 00:00:00.000000000 +0000 UTC, UTC
let dt = datetime::new(chrono::UTC, 0000, 1995, 09);

// long form
// 2038-01-19 03:14:07.000000000 +0000 UTC, UTC
let dta = datetime::new(chrono::UTC, 0000, 2038, 01, 19, 03, 14, 07, 000000000);

const Tokyo = &chrono::tz("Asia/Tokyo")!;

// 2038-01-19 12:14:07.000000000 +0900 JST, Asia/Tokyo
let dtb = datetime::new(
	Tokyo, 9 * time::HOUR,
	2038, 01, 19, 12, 14, 07, 000000000,
	// Notice the ^^ hour is as observed in Tokyo
);

assert(datetime::eq(dta, dtb)) // datetimes are equivalent

// current system time
let now = datetime::now();
```

Internally, for the simple case of a UTC datetime, the values are
submitted as-is for conversion to a moment. However, for non-normal
timezones (ones with offsets other than 0), the library will do a little
adjustment before converting.

Datetimes can exhibit different chronological values under different
timezones. The underlying moment values stay the same, but the cached
calendar and wall-time values adapt.

```hare
fn show(ls: []datetime::datetime) void = for (let i = 0z; i < len(ls); i += 1) {
	let dt = ls[i];
	fmt::println(
		// EMAILZ == "%a, %d %b %Y %H:%M:%S %z %Z"
		datetime::asformat(datetime::EMAILZ, &dt)!,
		"\t", dt.loc.name,
	)!;
};

let dt = datetime::new(chrono::UTC, 0000, 1995, 09, 24, 01, 00, 00, 000000000)!;
show([
	datetime::in(&chrono::tz("Pacific/Honolulu")!, dt),
	datetime::in(&chrono::tz("America/New_York")!, dt),
	datetime::in(&chrono::tz("Europe/Amsterdam")!, dt),
	datetime::in(&chrono::tz("Asia/Kathmandu")!, dt),
	datetime::in(&chrono::tz("Asia/Tokyo")!, dt),
	datetime::in(&chrono::tz("UTC")!, dt),
]);
```

This is the same datetime, the same moment in time, under different
timezones. Nepal's quirky UTC offset is handled just fine.

```
Sat, 23 Sep 1995 15:00:00 -1000 HST      Pacific/Honolulu
Sat, 23 Sep 1995 21:00:00 -0400 EDT      America/New_York
Sun, 24 Sep 1995 02:00:00 +0100 CET      Europe/Amsterdam
Sun, 24 Sep 1995 06:45:00 +0545 +0545    Asia/Kathmandu
Sun, 24 Sep 1995 10:00:00 +0900 JST      Asia/Tokyo
Sun, 24 Sep 1995 01:00:00 +0000 UTC      UTC
```

Datetimes handle timezone transitions too.

```hare
const Amst = &chrono::tz("Europe/Amsterdam")!;

// In Europe/Amsterdam, on March 26th, 1995
// the clock jumps forward 1 hour at 02:00 CET.
fmt::println("CET -> CEST")!;
show([
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 03, 26, 00, 00)!),
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 03, 26, 00, 30)!),
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 03, 26, 01, 00)!), // transition
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 03, 26, 01, 30)!),
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 03, 26, 02, 00)!),
]);

// In Europe/Amsterdam, on September 24th, 1995
// the clock jumps back 1 hour at 03:00 CEST.
fmt::println("CEST -> CET")!;
show([
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 09, 24, 00, 00)!),
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 09, 24, 00, 30)!),
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 09, 24, 01, 00)!), // transition
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 09, 24, 01, 30)!),
	datetime::in(Amst, datetime::new(chrono::UTC, 0, 1995, 09, 24, 02, 00)!),
]);
```

Notice at the change in time, offset, and zone abbreviation.

```
CET -> CEST
Sun, 26 Mar 1995 01:00:00 +0100 CET      Europe/Amsterdam
Sun, 26 Mar 1995 01:30:00 +0100 CET      Europe/Amsterdam
Sun, 26 Mar 1995 03:00:00 +0200 CEST     Europe/Amsterdam
Sun, 26 Mar 1995 03:30:00 +0200 CEST     Europe/Amsterdam
Sun, 26 Mar 1995 04:00:00 +0200 CEST     Europe/Amsterdam
CEST -> CET
Sun, 24 Sep 1995 02:00:00 +0200 CEST     Europe/Amsterdam
Sun, 24 Sep 1995 02:30:00 +0200 CEST     Europe/Amsterdam
Sun, 24 Sep 1995 02:00:00 +0100 CET      Europe/Amsterdam
Sun, 24 Sep 1995 02:30:00 +0100 CET      Europe/Amsterdam
Sun, 24 Sep 1995 03:00:00 +0100 CET      Europe/Amsterdam
```

It's important to distinguish "timezones" from "zones". If timezones are
a region, a zone is a sort of "state" which a timezone observes (CET,
CEST, etc.), depending on the date & time. Each timezone has a set of
zones it observes. It's also why datetime::new() asks for a zone offset,
to disambiguate between possible zones. I told you it was
complicated.

```hare
// A timezone; a political region with a ruleset regarding offsets for
// calculating localized civil time
type timezone = struct {
	// The textual identifier ("Europe/Amsterdam")
	name: str,

	// The base timescale (chrono::utc)
	timescale: *timescale,

	// The duration of a day in this timezone (24 * time::HOUR)
	daylength: time::duration,

	// The possible temporal zones a locality with this timezone can observe
	// (CET, CEST, ...)
	zones: []zone,

	// The transitions between this timezone's zones
	transitions: []transition,

	// A timezone specifier in the POSIX "expanded" TZ format.
	// See https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap08.html
	//
	// Used for extending calculations beyond the last known transition.
	posix_extend: str,
};

// A timezone state, with an offset for calculating localized civil time
type zone = struct {
	// The offset from the normal timezone (2 * time::HOUR)
	zoffset: time::duration,

	// The full descriptive name ("Central European Summer Time")
	name: str,

	// The abbreviated name ("CEST")
	abbr: str,

	// Indicator of Daylight Saving Time
	dst: bool,
};
```


## Formatting and parsing

In the real world, datetimes often exist as representations, and we
parse and format these representations. We have a family of functions
and predefined layouts centred around the [datetime::format][format]
function.

```hare
let dt = datetime::new(chrono::UTC, 0, 2038, 01, 19, 03, 14, 07)!;
fmt::println(datetime::asformat("%Y-%m-%d %H:%M:%S.%N %z %Z", &dt)!)!;
fmt::println(datetime::asformat(datetime::STAMP_NANO, &dt)!)!;
fmt::println(datetime::asformat(datetime::EMAILZ, &dt)!)!;
fmt::println(datetime::asformat(datetime::POSIX, &dt)!)!;
```

Output:

```
2038-01-19 03:14:07.000000000 +0000 UTC
2038-01-19 03:14:07.000000000
Tue, 19 Jan 2038 03:14:07 +0000 UTC
Tue Jan 19 03:14:07 UTC 2038
```

Parsing is a bit trickier. Hare holds the guarantee that datetimes are
internally valid and consistent, so we validate at the moment of
parsing. One can use the [datetime::from\_str][str] function for an
immediate parse if there's sufficient information given, or an immediate
error.

```hare
// 2038-01-19 00:00:00.000000000 UTC
let dt = datetime::from_str("%Y-%m-%d", "2038-01-19")!;
```

However, Hare provides a more delicate, progressive way of parsing with
the [datetime::builder][builder] type. This is internally just a
datetime, but waives the guarantee of validity, allowing its fields to
hold possibly incorrect values. The user gradually builds a datetime
with incremental calls to [datetime::parse][parse] and other
modifications, then attempts to manifest a real, valid datetime with
[datetime::finish][finish].

```hare
let mock = datetime::newbuilder();

datetime::parse(&mock, "Year:  %Y", "Year:  2038")!;
datetime::parse(&mock, "Month: %m", "Month: 01")!;
mock.day = 19;

let dt = match (datetime::finish(&mock, datetime::strategy::YMD)) {
case dt: datetime::datetime =>
	yield dt;
case datetime::insufficient =>
	fmt::println("Not enough information")!;
case datetime::invalid =>
	fmt::println("Datetime is invalid")!;
};
```


## Datetime arithmetic

Using what we've covered so far, we're able to perform simple arithmetic
using instants.

```hare
let dta = datetime::new(chrono::UTC, 0, 2000, 01, 01)!;
let dtb = datetime::new(chrono::UTC, 0, 2000, 01, 02)!;

let a = datetime::to_instant(dta);
let b = datetime::to_instant(dtb);
show([
	datetime::from_instant(a, chrono::UTC),
	datetime::from_instant(b, chrono::UTC),
]);

let delta = time::diff(a, b); // 86400 seconds (~1 day)
fmt::println("delta:", delta / time::SECOND)!;

let c = time::add(b,
	- 1 * time::HOUR
	+ 20 * time::MINUTE
	- 500 * time::MILLISECOND,
);
show([
	datetime::from_instant(c, chrono::UTC),
]);

fmt::printfln("a: {}.{}", a.sec, a.nsec)!;
fmt::printfln("b: {}.{}", b.sec, b.nsec)!;
fmt::printfln("c: {}.{}", c.sec, c.nsec)!;
```

Output:

```
Sat, 01 Jan 2000 00:00:00 +0000 UTC      UTC
Sun, 02 Jan 2000 00:00:00 +0000 UTC      UTC
delta: 86400
Sat, 01 Jan 2000 23:19:59 +0000 UTC      UTC
a: 946684800.0
b: 946771200.0
c: 946768799.500000000
```

However, to add and subtract datetimes, the datetime module provides
some basic functionality. We introduce the [datetime::period][period]
type to reason about calendrical differences between datetimes, as
opposed to "absolute" differences between instants.

```hare
// Represents a span of time in the proleptic Gregorian calendar, using relative
// units of time. Used for calendar arithmetic.
type period = struct {
	eras: int,
	years: int,
	// Can be 28, 29, 30, or 31 days long
	months: int,
	// Weeks start on Monday
	weeks: int,
	days: int,
	hours: int,
	minutes: int,
	seconds: int,
	nanoseconds: i64,
};
```

We can find the difference between datetimes.

```hare
let a = datetime::new(chrono::UTC, 0, 1982, 06, 25, 20, 19)!;
let b = datetime::new(chrono::UTC, 0, 2017, 10, 03, 20, 49)!;

// years:35 months:3 days:8 hours:0 minutes:30 seconds:0 nanoseconds:0
let p = datetime::diff(a, b);
```

Periods are *not* durations; years and months contain variable numbers
of days, and the length of a day varies by timezone transitions and leap
seconds. Datetime arithmetic is inherently highly irregular. With so
much complexity, it is the weakest point of the library (the same is
true across languages), but Hare tries to provide something sensible.
Third-party libraries, like a scheduling library, are expected to
provide their own comprehensive set of operators to suit their needs.

Further datetime-based arithmetic is under development. We have "add" and "hop"
functions which allow users to manipulate dates in the context of the calendar,
such as finding the date one year from today, or finding the datetime at the
start of last month. These functions still require some refinement, so we'll
save the introduction for later.

---

We wish you well in your datetime endeavours. If you'd like to discuss
or contribute to chronology in Hare, [reach us on IRC](/community/), or
write to us at:

- "Byron Torres" <b@torresjrjr.com>
- "Vlad-Stefan Harbuz" <vlad@vladh.net>

Happy time-travelling.


  [^mars]: Authur David Olson: "Although the tz database does not
  support time on other planets, it is documented here in the hopes that
  support will be added eventually." &mdash;
  https://data.iana.org/time-zones/theory.html#planets


  [t]: https://docs.harelang.org/time
  [e]: https://docs.harelang.org/time#error
  [i]: https://docs.harelang.org/time#instant
  [c]: https://docs.harelang.org/time/chrono
  [m]: https://docs.harelang.org/time/chrono#moment
  [tz]: https://docs.harelang.org/time/chrono#timezone
  [ts]: https://docs.harelang.org/time/chrono#timescale
  [tai]: https://docs.harelang.org/time/chrono#tai
  [utc]: https://docs.harelang.org/time/chrono#utc
  [mtc]: https://docs.harelang.org/time/chrono#mtc
  [tzfn]: https://docs.harelang.org/time/chrono#tz
  [LOCAL]: https://docs.harelang.org/time/chrono#LOCAL
  [dt]: https://docs.harelang.org/datetime
  [new]: https://docs.harelang.org/datetime#new
  [dtdt]: https://docs.harelang.org/datetime#datetime
  [parse]: https://docs.harelang.org/datetime#parse
  [format]: https://docs.harelang.org/datetime#format
  [str]: https://docs.harelang.org/datetime#from_str
  [builder]: https://docs.harelang.org/datetime#builder
  [finish]: https://docs.harelang.org/datetime#finish
  [period]: https://docs.harelang.org/datetime#period

  [taiw]: https://en.wikipedia.org/wiki/International_Atomic_Time
  [utcw]: https://en.wikipedia.org/wiki/Coordinated_Universal_Time
  [mtcw]: https://en.wikipedia.org/wiki/Coordinated_Mars_Time
  [utc0]: https://en.wikipedia.org/wiki/UTC%C2%B100:00
  [8601]: https://en.wikipedia.org/wiki/ISO_8601
  [long]: https://en.wikipedia.org/wiki/Mesoamerican_Long_Count_calendar

  [tzdb]: https://data.iana.org/time-zones/
  [tzif]: https://datatracker.ietf.org/doc/html/rfc8536

