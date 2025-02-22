# HyStartPP
## Implementation of HyStart++ in the Linux kernel's TCP CUBIC congestion control.

This project implements HyStart++ (as described in [RFC 9406](https://datatracker.ietf.org/doc/rfc9406)) within the TCP CUBIC congestion control algorithm in the Linux kernel (i.e., in `tcp_cubic.c`).
While HyStart++ is designed to replace its predecessor HyStart, we have currently retained both algorithms, allowing users to choose between them.

## Implementation Details
To enhance clarity and maintainability, we have defined a distinct set of variables for HyStart++ rather than reusing existing HyStart variables. For instance, when tracking the minimum RTT in the current round, we opted not to reuse HyStart's `curr_rtt`. Instead, we introduced `hspp_current_round_minrtt`, which serves the same purpose but ensures better separation between HyStart and HyStart++ logic.
This separation also makes it easier to eventually remove the HyStart implementation if needed.

The implementation includes `RFC9406_Lnnn` markers, where `nnn` is a three-digit number referencing the corresponding line in RFC 9406 [see here](./implementation/rfc9406.txt).

To provide clarity, we included a [block diagram](./implementation/block_diagram.pdf) that introduces the new procedures and code added to `tcp_cubic.c` and illustrates their interactions.

The [patch](./implementation/tcp_cubic.patch), and the modified `tcp_cubic.c` can be found in the `implementation`  directory within the project.

## Testing and Evaluation
We have conducted preliminary tests to evaluate the performance of HyStart++ in TCP CUBIC. The results, along with discussions and analysis, can be found [here](https://sussdeveloper.github.io/HyStartPP/evaluation/index.html).


## Enabling HyStart or HyStart++ and Running a Test
1. Selecting the Algorithm

By default, HyStart is enabled. To switch between HyStart and HyStart++, use the following commands:

Enable HyStart++ (disabling HyStart automatically):
<pre>
echo 1 > /sys/module/tcp_cubic/parameters/hystartpp
</pre>

Revert to HyStart (disable HyStart++):
<pre>
echo 0 > /sys/module/tcp_cubic/parameters/hystartpp
</pre>

2. Running a Test

After enabling the desired algorithm, you can conduct network performance tests.
By default, the source port used for TCP data transfer is <b>80</b>. If your test uses a different source port, you can set it using the following command:

<pre>
echo <port_number> > /sys/module/tcp_cubic/parameters/hystartpp_source_port
</pre>

This setting enables logging of information in the `/var/log/kern.log` file on Linux.

Foe example, the following log entry:
<pre>
t 874877310 c 22 i 21 f 0 r 100949 a 1 d 18824 l 0
</pre>
is from `/var/log/kern.log`, where:
- **t** represents the timestamp in microseconds.
- **c** represents the size of cwnd.
- **i** represents the number of packets in flight.
- **f** represents the flag (see the patch).
- **r** represents the smoothed RTT in microseconds.
- **a** represents the number of packets acknowledged by the received ACK.
- **d** represents the amount of data delivered so far.
- **l** represents the number of packet losses since the start of data transfer.
