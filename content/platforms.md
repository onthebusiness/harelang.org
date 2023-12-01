---
title: Supported platforms
---

Hare (will) support a variety of platforms. Adding new (Unix-like) platforms and
architectures is relatively straightforward.

Hare does not, and will not, support any proprietary operating systems.

## Architectures

<table>
  <thead>
    <tr>
      <th>Architecture</th>
      <th>Support status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>x86_64</td>
      <td><span class="yes">✓</span> Supported</td>
    </tr>
    <tr>
      <td>aarch64</td>
      <td><span class="yes">✓</span> Supported</td>
    </tr>
    <tr>
      <td>riscv64</td>
      <td><span class="yes">✓</span> Supported</td>
    </tr>
    <tr>
      <td>ppc64le</td>
      <td><span class="todo">…</span> Planned</td>
    </tr>
    <tr>
      <td>i686</td>
      <td><span class="todo">…</span> Planned</td>
    </tr>
    <tr>
      <td>arm32</td>
      <td><span class="todo">…</span> Planned</td>
    </tr>
  </tbody>
</table>

## Operating systems

<table>
  <thead>
    <tr>
      <th>Operating system</th>
      <th>Support status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Linux</td>
      <td><span class="yes">✓</span> Supported</td>
    </tr>
    <tr>
      <td>FreeBSD</td>
      <td><span class="yes">✓</span> Supported</td>
    </tr>
    <tr>
      <td>OpenBSD</td>
      <td><span class="yes">✓</span> Supported</td>
    </tr>
    <tr>
      <td>NetBSD</td>
      <td><span class="todo">…</span> In progress</td>
    </tr>
    <tr>
      <td>Illumos</td>
      <td><span class="todo">…</span> Planned</td>
    </tr>
    <tr>
      <td>Haiku</td>
      <td><span class="todo">…</span> Planned</td>
    </tr>
    <tr>
      <td>Plan 9</td>
      <td><span class="todo">…</span> Planned</td>
    </tr>
  </tbody>
</table>

Hare can also run on the bare metal, without a host operating system.

## Third-party support

Third parties may provide support for additional platforms. These ports are not
supported by Hare upstream. Do not file tickets, write to the mailing lists, or
ask questions on IRC related to these ports; see each port's own resources for
support.

- [macOS](https://github.com/hshq/harelang)
