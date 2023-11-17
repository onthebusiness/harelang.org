---
title: Martian time in Hare
author: Byron Torres
date: 2022-08-01
---


> This is a republication from [Byron Torres][or]'s blog.


## Preface

Development of the [Hare programming langauge][ha] is chugging along.
From what I've gathered from IRC, Drew Devault has been spending most of
his hare-dev time on [Helios][he], in an effort to throw real code at
Hare and scrutinize its design choices. For example, since then, a
`@threadlocal` keyword has come into consideration.

My focus area is naturally the [datetime][dt] and [time::chrono][tc]
modules of the standard library, both of which I primarily wrote, and
which I maintain and intend to for a long time. These modules too have
been made with unique design choices.

To test the robustness and modularity of *datetime* and *time::chrono*,
I've been working on [hare-mbc][lib], a Hare library which provides a
Martian chronology based upon the [Martian Business Calendar][mbc] by
Bruce Mills. I've talked about chronologies, *datetime* and
*time::chrono* in some depth already at the Hare blog, so I'll try not
to repeat much here.

> Chronology in Hare  
> <https://harelang.org/blog/2022-04-17-chronology-in-hare/>

I decided to implement the Martian Business Calendar over other more
commonly known ones like the Darian, Utopian or Martiana calendars,
because I liked its use of leep weeks better than leap days, which does
not interrupt the week cycle and keeps the weekdays in the same position
in every month and year.

In the MBC chronology, Mars years have either 665 or 672 sols (Martian
days), about twice that of Earth's. Years have 24 months, each having 28
sols, except for the last month of the year, which skips an intercalary
week nearly every second year.

I found this design to be easier to implement than *datetime*'s
Gregorian calendar. The MBC calendar has a cycle of 76 years (50813
sols). Unlike *datetime* which uses a mathematical formula, I had
time::mbc use a lookup table to calculate the the number of sols upto a
given year in a cycle.


## Demo

Let's explore Martian timekeeping and chronology in Hare with hare-mbc.

> NOTE: this library is incomplete and untested. The following code uses
> a forked version of the standard library, which I suspect to be merged
> soon after review. Nonetheless, there are some interesting results
> here.

hare-mbc comes with the time::mbc module. Using our trusty haredoc tool,
we can get its documentation.

[$ haredoc -Fhtml time::mbc](https://torresjrjr.com/archive/2022-08-01-hare-martian-time/docs)

This module is based off the *datetime* module, though I left out a lot
of stuff like parsing, datetime arithmetic and pseudo-datetime
functionality, to keep the module small for the time being. Let's import
time::mbc and some other stdlib modules to play with.

```hare
use time;
use time::mbc;
use time::chrono;
use datetime;
use fmt;
```

Let's create a few MBC datetimes.

```hare
// MBC epoch
let epoc = mbc::new(chrono::MTC, 0)!;

// Unix epoch
let unix = mbc::new(chrono::MTC, 0, 0191,20,23, 07,06,01,057366670)!;

// My birthday
let birt = mbc::new(chrono::MTC, 0, 0207,11,16, 08,49,27,279563480)!;

// Hare's first commit.
let hare = mbc::new(chrono::MTC, 0, 0218,10,19, 09,20,53,344357297)!;
```

Notice we're using the stdlib's [chrono::MTC][mtcloc] locality
(Coordinated Mars Time), and thus using the [chrono::mtc][mtctsc]
timescale by proxy. We are also specifying an offset of `0` for all our
datetimes. We won't be experimenting with offseted timezones today,
though they should work just fine. Perhaps I'll add [Airy-0][airy] and
other such Martian timezones.

The declaration for `epoc` uses [mbc::new](https://torresjrjr.com/archive/2022-08-01-hare-martian-time/docs#new)'s short form, which
is equivilent to this:

```hare
let epoc = mbc::new(chrono::MTC, 0, 0000,01,01, 00,00,00,000000000)!;
```

The same moment on Earth occurs on the Gregorian date 1609 March 12th at
19:19:16 UTC. This is the [Telescopic epoch][tel], named after the fact
that Galileo Galilei first observed Mars around this time. *The Utopian
calendar* website details the [calculation of this date][epoch], which
the stdlib's [chrono::mtc timescale already does][src] in a similar
fashion.

Let's format and print these datetimes.

```hare
const buf: [64]u8 = [0...];

fmt::println(mbc::bsformat(buf, mbc::STAMP_NOZL, &epoc)!)!;
fmt::println(mbc::bsformat(buf, mbc::STAMP_NOZL, &unix)!)!;
fmt::println(mbc::bsformat(buf, mbc::STAMP_NOZL, &birt)!)!;
fmt::println(mbc::bsformat(buf, mbc::STAMP_NOZL, &hare)!)!;
fmt::println()!;
fmt::println(mbc::bsformat(buf, mbc::STELLAR, &epoc)!)!;
fmt::println(mbc::bsformat(buf, mbc::STELLAR, &unix)!)!;
fmt::println(mbc::bsformat(buf, mbc::STELLAR, &birt)!)!;
fmt::println(mbc::bsformat(buf, mbc::STELLAR, &hare)!)!;
fmt::println()!;
fmt::println(mbc::bsformat(buf, mbc::EMAIL_Z, &epoc)!)!;
fmt::println(mbc::bsformat(buf, mbc::EMAIL_Z, &unix)!)!;
fmt::println(mbc::bsformat(buf, mbc::EMAIL_Z, &birt)!)!;
fmt::println(mbc::bsformat(buf, mbc::EMAIL_Z, &hare)!)!;
fmt::println()!;
```

Output:

```
0000-01-01 00:00:00.000000000 +0000 MTC MTC
0191-20-23 07:06:01.057366670 +0000 MTC MTC
0207-11-16 08:49:27.279563480 +0000 MTC MTC
0218-10-19 09:20:53.344357297 +0000 MTC MTC

0000 Sagittarius 01, Mon 00:00 MTC
0191 Boötes 23, Tue 07:06 MTC
0207 Taurus 16, Tue 08:49 MTC
0218 Perseus 19, Fri 09:20 MTC

Mon, 01 Sgtr 0000 00:00:00 +0000 MTC
Tue, 23 Boot 0191 07:06:01 +0000 MTC
Tue, 16 Taur 0207 08:49:27 +0000 MTC
Fri, 19 Pers 0218 09:20:53 +0000 MTC
```

MBC optionally includes a Latin-based naming scheme for the 24 months
using constellations, 12 of which are from the zodiac. hare-mbc provides
these names, as well as IAU's 3-letter and NASA's 4-letter
[constellation abbreviations][abbr].

To simplify things, let's write a pretty printing helper function.

```hare
fn show(ls: (datetime::datetime | mbc::datetime)...) void = {
	for (let i = 0z; i < len(ls); i += 1) {
		match (ls[i]) {
		case let dt: datetime::datetime =>
			fmt::println(datetime::bsformat(buf,
				datetime::EMAILZ, &dt)!)!;
		case let dt: mbc::datetime =>
			fmt::println(mbc::bsformat(buf,
				mbc::EMAIL_Z, &dt)!)!;
		};
	};
	fmt::println()!;
};
```

Let's convert our Martian datetime `birt` to an Earthly datetime and
back. This is actually my birthday. Coincidentally, as you can see, I'm
a Taurus in two chronologies.

```hare
let earth = datetime::from_moment(
	chrono::in(chrono::UTC, *(&birt: *chrono::moment))
);
let mars = mbc::from_moment(
	chrono::in(chrono::MTC, *(&earth: *chrono::moment))
);
show(birt, earth, mars);
```

Output:

```
Tue, 16 Taur 0207 08:49:27 +0000 MTC
Thu, 13 May 1999 00:00:00 +0000 UTC
Tue, 16 Taur 0207 08:49:27 +0000 MTC
```

Here we are making a pointer to `birt`, casting it to a
[chrono::moment][mom] type, and dereferencing our new moment, which
[chrono::in][cin] accepts. We're using a special property of
[mbc::datetime](https://torresjrjr.com/archive/2022-08-01-hare-martian-time/docs#datetime) &mdash; the fact that it
embeds chrono::moment, just like the stdlib [datetime::datetime][dtdt] type.
Third-party libraries like hare-mbc are expected to create their own
datetime types like this, thus unifying all datetime types.

The chrono::moment type in turn embeds [time::instant][inst]. That is to
say, every datetime is really a time::instant with extra information,
notably a .loc field for the moment's [chrono::locality][loc], itself
containing a .timescale field for the locality's [chrono::timescale][tsc].

We ask chrono::in to change the locality of our Martian datetime from
MTC to UTC. Since these two localities use different timescales, a
conversion from one timescale to the other will occur, like the
following code. Hare uses [chrono::tai][tai] as an intermediary
timescale for conversions.

```hare
let mtc = *(&mars: *time::instant);
let utc = chrono::utc.from_tai(chrono::mtc.to_tai(mtc)!)!;
let earth = datetime::from_instant(chrono::UTC, utc);
show(earth);
```

Output:

```
Thu, 13 May 1999 00:00:00 +0000 UTC
```

MBC uses the common 24-hour clock to track time throughout a sol, just
like we do with days on earth. That's 86400 seconds per day/sol.
However, Martian-time runs ~3% slower than Earth-time, and we
account for this by linearly scaling Martian seconds when converting.

This also means adding a [time::duration][dur] to a datetime requires
knowledge of what timescale you're working with. For example:

```hare
let nextday_utc = datetime::from_instant(
	chrono::UTC,
	time::add(*(&earth: *time::instant), 24 * time::HOUR),
);

let mars_nextday = mbc::from_instant(
	chrono::MTC,
	time::add(*(&mars: *time::instant), 24 * time::HOUR),
);
let nextday_mtc = datetime::from_moment(
	chrono::in(chrono::UTC, *(&mars_nextday: *chrono::moment)),
);

show(nextday_utc, nextday_mtc);
```

Output:

```
Fri, 14 May 1999 00:00:00 +0000 UTC
Fri, 14 May 1999 00:39:35 +0000 UTC
```

Indeed, 24 hours in Martian-time is equal to 24 hours, 39 minutes, and
35 seconds in Earth-time.


## Conclusion

The library does have some warts, and the floating point arithmetic used
internally can sometimes cause slight inaccuracies with the nanosecond
(should be fixable). Nonetheless, I think Hare has proven quite capable
for handling foreign chronologies. We didn't just handle another
calendar system, but another underlying timescale too, and quite
seemlessly. This enabled us to model our world very accurately.

hare-mbc is to be refined and expanded, and be a blueprint for future
calendar libraries. We will need implementations for common chronologies
such as the Japanese, Hijrah, Hebrew, and Thai-Buddhist calendars, etc.
I also have [hare-mayan][may] awaiting in the lab, which uses a 5-unit
modified vigesimal calendar.

This experimental demo was fun, and I hope Hare can continue to provide
good timekeeping capabilities upto Hare 1.0 and beyond. Maybe even in
space, who knows?



  [or]: https://torresjrjr.com/archive/2022-08-01-hare-martian-time/
  [ha]: https://torresjrjr.com/archive/2022-05-02-welcome-harelang/

  [he]: https://drewdevault.com/2022/06/13/helios.html
  [mbc]: http://www.bdm.id.au/calendar/mars_calendar.html
  [tel]: https://ops-alaska.com/time/gangale_mst/darian.htm#Telescopic%20Epoch
  [epoch]: https://marscalendar.com/epoch

  [abbr]: https://en.wikipedia.org/wiki/IAU_designated_constellations
  [airy]: https://en.wikipedia.org/wiki/Airy-0

  [src]: https://git.sr.ht/~sircmpwn/hare/tree/master/item/time/chrono/timescale.ha#L162
  [lib]: https://git.sr.ht/~torresjrjr/hare-mbc
  [may]: https://git.sr.ht/~torresjrjr/hare-mayan

  [inst]: https://docs.harelang.org/time#instant
  [dur]: https://docs.harelang.org/time#duration
  [dt]: https://docs.harelang.org/datetime
  [dtdt]: https://docs.harelang.org/datetime#datetime
  [tc]: https://docs.harelang.org/time/chrono
  [mtcloc]: https://docs.harelang.org/time/chrono#MTC
  [mtctsc]: https://docs.harelang.org/time/chrono#mtc
  [tai]: https://docs.harelang.org/time/chrono#tai
  [mom]: https://docs.harelang.org/time/chrono#moment
  [cin]: https://docs.harelang.org/time/chrono#in
  [loc]: https://docs.harelang.org/time/chrono#locality
  [tsc]: https://docs.harelang.org/time/chrono#timescale
