# Lab 2: gNB TDD Traffic Analysis

## 1. Goal

In this lab, you will compare three 5-second UDP iperf3 tests. You will learn how TDD slot allocation can affect downlink and uplink performance.

You will also read the broadcast TDD configuration from SIB1 in Wireshark, draw the downlink/uplink time allocation, and relate the configuration to the iperf3 results.

You do not need to understand every packet. Follow the steps, complete the tables, and answer the short questions.

## 2. Test Scenarios and Files

| Scenario | TDD setting | Traffic direction | Main idea |
| --- | --- | --- | --- |
| A | DL-heavy | Downlink | The TDD setting favors the traffic direction |
| B | DL-heavy | Uplink | There are fewer opportunities for uplink traffic |
| C | Balanced | Uplink | More uplink opportunities are available than in B |

[Download the lab files from Google Drive](https://drive.google.com/drive/u/0/folders/1gTgosNkWFuak_sXbHFTxqaRqXb15Wz9T).

### Two Types of Packet Captures

Two types of packet captures are provided in this lab:

- **Configuration captures:** Recorded separately during initial cell access to show the broadcast SIB1 messages for the two TDD settings. Use these files to read and draw the common TDD configuration. Only two configuration captures are required because Scenarios A and B share the DL-heavy setting.
- **Scenario captures:** Recorded during the iperf3 tests. Use these files to analyze user-plane packets and the TCP control connection. Separate captures are provided for A, B, and C because each scenario uses a different combination of TDD setting and traffic direction.

The configuration captures are reference recordings for the corresponding settings, not simultaneous recordings of the original iperf3 runs. The scenario-to-configuration mapping below is supplied by the instructor; it cannot be established from the iperf3 JSON alone. Configuration captures are not used to calculate test throughput, loss, jitter, RTT, or packet intervals.

### Configuration Captures — Task 4 Only

| TDD setting | Applies to | Configuration capture |
| --- | --- | --- |
| DL-heavy | Scenarios A and B | `tdd-dl-heavy-sib1.pcapng` |
| Balanced | Scenario C | `tdd-balanced-sib1.pcapng` |

### Performance Files — Tasks 1–3 and 5–8

| Scenario | iperf3 JSON | Scenario capture |
| --- | --- | --- |
| A | `scenario-a-downlink.json` | `scenario-a-dl-heavy-downlink.pcapng` |
| B | `scenario-b-uplink.json` | `scenario-b-dl-heavy-uplink.pcapng` |
| C | `scenario-c-uplink.json` | `scenario-c-balanced-uplink.pcapng` |

> The JSON files report the traffic direction and performance results. They do not contain the gNB TDD configuration. Use the configuration captures for SIB1 and the scenario captures for traffic analysis.

## 3. Task 1 — Confirm the Traffic Direction

Open each JSON file and find:

```text
start → test_start → reverse
```

- `reverse: 1` means downlink in this lab.
- `reverse: 0` means uplink in this lab.

Complete the table:

| Scenario | Reverse value | Direction |
| --- | ---: | --- |
| A | 1 | Downlink |
| B | 0 | Uplink |
| C | 0 | Uplink |

**Question:** Does the traffic direction shown in the JSON match the scenario description? Show the proof.
- Yes, the traffic direction in the JSON files does match the scenario description for each scenario, we have in fact:
  - Scenario A => start → test_start → reverse : 1 => Downlink
    ![Scenario A Task 1](screenshots/scenarioAtask1.png)
  - Scenario B => start → test_start → reverse : 0 => Uplink
    ![Scenario B Task 1](screenshots/scenarioBtask1.png)
  - Scenario C => start → test_start → reverse : 0 => Uplink
    ![Scenario C Task 1](screenshots/scenarioCtask1.png)


## 4. Task 2 — Compare Throughput

In each JSON file, find:

```text
end → sum → bits_per_second
```

Convert the value to Mbps:

```text
Throughput (Mbps) = bits_per_second ÷ 1,000,000
```

Complete the table:

| Scenario | Direction | Throughput (Mbps) |
| --- | --- | ---: |
| A | Downlink | 90.654195836 |
| B | Uplink | 109.11724969 |
| C | Uplink | 124.41674913 |

Calculate how much C improves over B:

```text
Improvement (%) = (C - B) ÷ B × 100
```

Answer:

1. What is the downlink throughput in Scenario A?
   - It is 90.654195836 Mbps.
     ![Scenario A Task 2](screenshots/scenarioAtask2.png)
2. What is the uplink throughput in Scenario B?
   - It is 109.11724969 Mbps.
     ![Scenario B Task 2](screenshots/scenarioBtask2.png)
3. Is the uplink throughput higher in B or C?
   - It is higher in C than in B : 124.41674913 > 109.11724969.
     ![Scenario C Task 2](screenshots/scenarioCtask2.png)
4. By approximately what percentage does C improve over B?
   - It is approximately an improvement of 14.02% => (( 124.41674913−109.11724969)÷109.11724969)×100 = 14,021155668)

> Use the JSON `bits_per_second` value. Do not recalculate throughput using the other time fields.

## 5. Task 3 — Check the Trade-off

In each JSON file, find:

```text
end → sum → lost_percent
end → sum → jitter_ms
```

Complete the table:

| Scenario | Throughput (Mbps) | Packet loss (%) | Jitter (ms) |
| --- | ---: | ---: | ---: |
| B | 109.11724969 | 0.23565878306653645 | 0.11619665995310689 |
| C | 124.41674913  | 2.133919261135111 | 0.16204804777679407 |

Answer:

1. Does C have higher uplink throughput than B?
   - Yes, C have higher uplink throughput than B. 
2. Does C have lower packet loss than B?
   - No, C have higher packet loss percentage than B, approximately 10 times more (~2.13% compared to ~0.24%)
3. What trade-off, if any, is visible when changing from B to C?
   - Compared with Scenario B, Scenario C has higher throughput, higher packet loss, and higher jitter. These results suggest that ther are more Uplink scheduling opportunities

A simple answer format is:

> Compared with Scenario B, Scenario C has ______ throughput, ______ packet loss, and ______ jitter. These results suggest ______.

Use the measured values. If one result improves while another worsens, explain that trade-off. Do not assume that increasing UL resources always increases packet loss.

## 6. Task 4 — Read SIB1 and Draw the TDD Configuration

Use these **two configuration captures** for this task:

- `tdd-dl-heavy-sib1.pcapng`: Reference configuration for Scenarios A and B.
- `tdd-balanced-sib1.pcapng`: Reference configuration for Scenario C.

These files were recorded separately from the original iperf3 tests. Use them to examine the supplied reference settings.

### 4.1 What Is SIB1?

A gNB broadcasts system information so that a UE can learn how to access the cell.

- **MIB (Master Information Block):** Provides basic information, including parameters needed to find SIB1.
- **SIB1 (System Information Block Type 1):** Provides cell-access and common radio configuration information. In this TDD lab, it includes `tdd-UL-DL-ConfigurationCommon`.

You will read the **common TDD configuration in SIB1**, not decode every MIB/SIB field. This configuration describes DL, UL, and flexible time resources; it is not a record of the actual data scheduled in every slot.

### 4.2 Enable MAC-NR Decoding and Find SIB1

These captures contain NR MAC messages carried over UDP. **Enable MAC-NR-over-UDP decoding before applying the SIB1 filter.** Without this decoder, the packets may appear only as UDP and the SIB1 filter may return no results.

1. Open `tdd-dl-heavy-sib1.pcapng` in Wireshark.
2. Open **Analyze → Enabled Protocols**. Locate **MAC-NR over UDP** (`mac_nr_udp`) in the heuristic dissectors list and enable it. Depending on the Wireshark version, the heuristic list may be shown separately. Apply the change and reload the capture if needed.
3. Apply this display filter:

   ```text
   nr-rrc.systemInformationBlockType1_element
   ```

4. Select a SIB1 packet. In the packet details, expand:

   ```text
   NR Radio Resource Control (RRC) protocol
     → BCCH-DL-SCH-Message
       → message
         → c1
           → systemInformationBlockType1
             → servingCellConfigCommon
               → tdd-UL-DL-ConfigurationCommon
   ```

5. Read `referenceSubcarrierSpacing`, then expand `pattern1` and record:

   ```text
   dl-UL-TransmissionPeriodicity
   nrofDownlinkSlots
   nrofDownlinkSymbols
   nrofUplinkSlots
   nrofUplinkSymbols
   ```

6. Check whether `pattern2` is also present.
7. Repeat for `tdd-balanced-sib1.pcapng`.

Record the **decoded labels and units** shown in Wireshark. For example, record `30 kHz` or `5 ms`, not the enumeration number displayed in parentheses.

You can also locate packets containing the TDD configuration with:

```text
nr-rrc.tdd_UL_DL_ConfigurationCommon_element
```

Confirm that the selected TDD fields are inside **SIB1**. If a filter is not recognized in your version, try `nr-rrc` and locate `systemInformationBlockType1` manually.

> **Troubleshooting:** If packets still appear only as UDP, check that the **MAC-NR over UDP heuristic dissector** is enabled, rather than only the MAC-NR protocol. If the file still does not decode, ask the instructor to check the settings. Do not infer TDD parameters from packet timing. Graphical menu labels may vary by version.

### 4.3 Record the TDD Parameters

Complete the table for `pattern1`:

| Item | DL-heavy: A/B | Balanced: C |
| --- | --- | --- |
| Configuration capture filename | tdd-dl-heavy-sib1.pcapng | tdd-balanced-sib1.pcapng |
| SIB1 packet number | 2 | 2 |
| `referenceSubcarrierSpacing` (kHz) | 30 | 30 |
| `dl-UL-TransmissionPeriodicity` (ms) | 5 | 2.5 |
| `nrofDownlinkSlots` | 7 | 2 |
| `nrofDownlinkSymbols` | 6 | 6 |
| `nrofUplinkSlots` | 2 | 2 |
| `nrofUplinkSymbols` | 4 | 4 |
| Is `pattern2` present? | No | No |

If `pattern2` is present, add another table containing its period and four slot/symbol counts. It uses the same reference SCS.

Take **one screenshot per configuration** showing the selected SIB1 packet and the expanded TDD fields. Include `pattern2` in an additional screenshot if it is present and does not fit.
- TDD DL Heavy SIB1
  ![TDD DL Heavy SIB1](screenshots/task4heavy.png)
- TDD Balanced SIB1
  ![TDD balanced SIB1](screenshots/task4balanced.png)

### 4.4 Draw the TDD Time Allocation

Draw one diagram for DL-heavy and one for balanced. **Use the same 0–5 ms time axis for both diagrams.** Derive each repeating period from SIB1 and repeat the shorter pattern as needed to fill the 5 ms interval. Mark every repeating-period boundary.

A spreadsheet, a drawing tool, or a hand-drawn diagram is acceptable. Include the diagrams as images in your report.

For the normal cyclic prefix used in this exercise, each reference slot contains **14 OFDM symbols**, numbered **0–13**. Use the SIB1 **reference SCS** to construct the common pattern:

| Reference SCS | Reference slot duration |
| --- | ---: |
| 15 kHz | 1 ms |
| 30 kHz | 0.5 ms |
| 60 kHz | 0.25 ms |
| 120 kHz | 0.125 ms |

```text
Number of reference slots in a pattern = Pattern period ÷ Reference slot duration
```

For each pattern:

1. Place `nrofDownlinkSlots` full DL slots at the beginning.
2. If `nrofDownlinkSymbols` is nonzero, mark that many symbols at the beginning of the next slot as DL.
3. Place `nrofUplinkSlots` full UL slots at the end.
4. If `nrofUplinkSymbols` is nonzero, mark that many symbols at the end of the slot immediately before the full UL slots as UL. If there are no full UL slots, use the last slot of the pattern.
5. Mark all remaining symbols as **F (flexible)**. The partial DL and partial UL portions may be in the same slot or in different slots.

Use **D = downlink**, **U = uplink**, and **F = flexible**. Include a legend, the common 0–5 ms time scale, slot indices, and repeating-period boundaries. You may restart the local slot index at zero at each period boundary, but label the boundaries clearly. Expand any mixed slot to show all 14 symbols. Do not automatically label flexible symbols as UL or guard time.

If `pattern2` is present, draw `pattern1` followed by `pattern2`; the combined sequence repeats. Label each pattern boundary and show the full combined duration.

#### TDD Allocation Diagrams (0–5 ms)

**Legend:**
- `D`: Downlink symbol/slot
- `U`: Uplink symbol/slot
- `F`: Flexible symbol
- `M`: Mixed slot (contains D, F, and U symbols)

**1. DL-heavy Configuration (Period = 5 ms, 1 repetition in 5 ms):**
```text
Time (ms)  : 0.0                                                                    5.0
Slot index : |  0  |  1  |  2  |  3  |  4  |  5  |  6  |     7 (M)     |  8  |  9  |
Slot type  : |  D  |  D  |  D  |  D  |  D  |  D  |  D  | 6D | 4F | 4U  |  U  |  U  |
             |<--------------------------- Pattern 1 (5 ms) ----------------------->|
```

*Mixed Slot 7 symbol details (14 OFDM symbols):*
```text
Symbol index : | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
Direction    : | D | D | D | D | D | D | F | F | F | F |  U |  U |  U |  U |
```

**2. Balanced Configuration (Period = 2.5 ms, 2 repetitions in 5 ms):**
```text
Time (ms)  : 0.0                       2.5 (Boundary)                  5.0
Slot index : |  0  |  1  |    2 (M)    |  3  |  4  |  0  |  1  |    2 (M)    |  3  |  4  |
Slot type  : |  D  |  D  | 6D | 4F | 4U|  U  |  U  |  D  |  D  | 6D | 4F | 4U|  U  |  U  |
             |<------- Repetition 1 (2.5 ms) ----->|<------- Repetition 2 (2.5 ms) ----->|
```

*Mixed Slots (Slot 2 in each repetition) symbol details (14 OFDM symbols):*
```text
Symbol index : | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
Direction    : | D | D | D | D | D | D | F | F | F | F |  U |  U |  U |  U |
```

---

First, count the symbols over **one complete repeating sequence** for each configuration. The two sequences may have different durations, so their raw UL symbol counts alone cannot be used to compare resources per unit time:

```text
Configured DL symbols = 14 × Full DL slots + Partial DL symbols
Configured UL symbols = 14 × Full UL slots + Partial UL symbols
Flexible symbols = Total symbols - Configured DL symbols - Configured UL symbols
Configured UL symbol share (%) = Configured UL symbols ÷ Total symbols × 100
```

For two patterns, sum the counts and durations of both patterns before calculating the share. These are symbol-count shares of the common configuration, not exact airtime measurements or measured throughput shares.

| TDD setting | Repeating duration (ms) | Total reference slots | DL symbols | UL symbols | Flexible symbols | Configured UL symbol share (%) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| DL-heavy | 5 | 10 | 104 | 32 | 4 | 22.86% |
| Balanced | 2.5 | 5 | 34 | 32 | 4 | 45.71% |

Next, count the symbols over the **same 5 ms interval** shown in your diagrams. Multiply each per-sequence count by the number of complete repetitions in 5 ms. For these captures, 5 ms contains a whole number of repetitions for both configurations.

| TDD setting | Repeating duration (ms) | Repetitions in 5 ms | DL symbols in 5 ms | UL symbols in 5 ms | Flexible symbols in 5 ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| DL-heavy | 5 | 1 | 104 | 32 | 4 |
| Balanced | 2.5 | 2 | 68 | 64 | 8 |

**Question:** Can two patterns have the same UL symbol count per period but provide different numbers of UL symbols per unit time? Explain using your two tables.

**Answer:**
Yes. As shown in the first table, both configurations have exactly 32 UL symbols per repeating period. However, the Balanced configuration has a shorter period (2.5 ms) compared to DL-heavy (5 ms). Consequently, as shown in the second table, the Balanced pattern repeats twice as often over the same 5 ms interval, providing 64 UL symbols in 5 ms compared to only 32 UL symbols for DL-heavy.

### 4.5 Connect SIB1 to the Performance Results

Use the supplied mapping between the reference configuration captures and scenarios. Complete this table using Tasks 1–3 and your SIB1 analysis:

| Scenario | Reference TDD setting | JSON traffic direction | Configured UL symbol share (%) | Throughput (Mbps) | Packet loss (%) |
| --- | --- | --- | ---: | ---: | ---: |
| A | DL-heavy | Downlink | 22.86% | 90.654195836 | 0.442954627080091 |
| B | DL-heavy | Uplink | 22.86% | 109.11724969 | 0.23565878306653645 |
| C | Balanced | Uplink | 45.71% | 124.41674913 | 2.133919261135111 |

Answer:

1. Which decoded TDD parameters differ between DL-heavy and balanced?
   - The parameters that differ are `dl-UL-TransmissionPeriodicity` (5 ms for DL-heavy vs. 2.5 ms for Balanced) and `nrofDownlinkSlots` (7 slots for DL-heavy vs. 2 slots for Balanced). The other parameters (`referenceSubcarrierSpacing`, `nrofDownlinkSymbols`, `nrofUplinkSlots`, and `nrofUplinkSymbols`) remain identical.

2. Which configuration provides more configured UL symbols per unit time? Support your answer with the diagram and calculated share.
   - The Balanced configuration provides more configured UL symbols per unit time. As shown in the diagrams and calculated tables, Balanced provides 64 UL symbols per 5 ms interval (a 45.71% configured UL share) compared to only 32 UL symbols per 5 ms interval (a 22.86% configured UL share) for the DL-heavy configuration.

3. How does that difference help explain the B/C throughput comparison?
   - Scenario C achieves higher uplink throughput (124.42 Mbps) than Scenario B (109.12 Mbps) because the Balanced configuration allocates twice as many UL transmission opportunities per unit time (64 UL symbols in 5 ms vs. 32 in DL-heavy). This higher density of uplink slots reduces buffering delays and accommodates more uplink data transmissions.

4. Why can A and B use the same SIB1 TDD configuration even though their traffic directions differ?
   - SIB1 broadcasts a common cell-level TDD frame structure (`tdd-UL-DL-ConfigurationCommon`) that defines the duplexing time slots for both downlink and uplink across the entire cell. The gNB dynamic MAC scheduler can then allocate available DL resources to DL traffic (as in Scenario A) or available UL resources to UL traffic (as in Scenario B) independently within that shared TDD framework.

> More configured UL resources can provide more opportunities for uplink traffic, but they do not guarantee a proportional throughput increase. Scheduling, radio conditions, overhead, offered traffic, and other radio settings also affect the result. This comparison does not establish that the TDD pattern is the only cause of a performance difference. SIB1 does not show actual slot-by-slot data scheduling. Because the configuration captures were recorded separately, use them as references for the supplied settings, not as independent proof of what was broadcast during each original performance run.

### Optional Observation — PRACH Configuration Index

In the same SIB1, find:

```text
servingCellConfigCommon
  → uplinkConfigCommon
    → initialUplinkBWP
      → rach-ConfigCommon
        → setup
          → rach-ConfigGeneric
            → prach-ConfigurationIndex
```

Wireshark may omit or display the `setup` layer differently. You can also use this display filter, then confirm the field is inside SIB1:

```text
nr-rrc.prach_ConfigurationIndex
```

| TDD setting | SIB1 packet number | PRACH configuration index |
| --- | ---: | ---: |
| DL-heavy | 2 | 98 |
| Balanced | 2 | 98 |

PRACH is used to send a random-access preamble. Its configuration index is not the DL/UL slot ratio and does not directly explain iperf3 throughput. Equal indices in the two configurations are a valid observation. Drawing PRACH occasions requires additional parameters and the applicable TS 38.211 table; it is outside the required TDD drawing task.

## 7. Task 5 — Is the TDD Setting Suitable?

Complete the table using the supplied scenario mapping, the SIB1 reference configurations from Task 4, and your throughput results:

| Scenario | TDD setting | Traffic direction | Is the setting suitable for this direction? Why? |
| --- | --- | --- | --- |
| A | DL-heavy | Downlink | Yes. The DL-heavy pattern allocates the vast majority of time resources to downlink (104 DL symbols vs. 32 UL symbols per 5 ms), providing sufficient transmission opportunities for downlink traffic. |
| B | DL-heavy | Uplink | No. The DL-heavy pattern allocates only 22.86% of symbols to uplink (32 UL symbols in 5 ms), which severely restricts uplink transmission opportunities and throughput under heavy uplink demand. |
| C | Balanced | Uplink | Yes. The Balanced configuration provides twice as many uplink opportunities per unit time (64 UL symbols per 5 ms, 45.71% share) compared to DL-heavy, accommodating uplink traffic much more effectively. |

Use these ideas:

- More DL resources provide more downlink transmission opportunities.
- More UL resources provide more uplink transmission opportunities.
- A DL-heavy setting is suitable for downlink traffic.
- A DL-heavy setting may limit heavy uplink traffic.
- A balanced setting gives uplink traffic more opportunities than the DL-heavy setting in this lab.

> The configuration captures show the reference broadcast TDD settings. The iperf3 JSON reports performance; it does not contain TDD parameters. The scenario captures are used for traffic analysis and need not include SIB1.

**Question:** Why might Scenario C achieve better uplink throughput than Scenario B? Refer to your SIB1 diagrams and measured results.

**Answer:**
Scenario C achieves higher uplink throughput (~124.42 Mbps vs. ~109.12 Mbps in B) because the Balanced TDD configuration doubles the uplink transmission opportunities per unit time. As derived from SIB1, the Balanced configuration has a 2.5 ms periodicity (repeating twice in 5 ms) providing 64 UL symbols per 5 ms (45.71% share), whereas the DL-heavy configuration in Scenario B has a 5 ms periodicity providing only 32 UL symbols per 5 ms (22.86% share). The more frequent UL allocation reduces buffering delay at the UE and grants more airtime for sending UDP packets, directly leading to higher measured throughput.

## 8. Task 6 — Find GTP-U Information

Open a **scenario capture** in Wireshark and enter this display filter:

```text
gtp && ip
```

Choose a packet that contains two IPv4 headers. In the packet details, expand:

1. the first `Internet Protocol Version 4` header;
2. `User Datagram Protocol`;
3. `GPRS Tunneling Protocol`; and
4. the second `Internet Protocol Version 4` header.

Record the following information from one packet:

| Item | Value |
| --- | --- |
| Scenario capture filename | scenario-a-dl-heavy-downlink.pcapng | scenario-b-dl-heavy-uplink.pcapng | scenario-c-balanced-uplink.pcapng |
| Packet number | 1831 | 2065 | 1430 |
| Outer source IP | 192.168.70.129 | 192.168.70.129 | 192.168.70.129 |
| Outer destination IP | 192.168.70.134 | 192.168.70.134 | 192.168.70.134 |
| Inner source IP | 10.0.0.6 | 10.0.0.6 | 10.0.0.6 |
| Inner destination IP | 192.168.70.135 | 192.168.70.135 | 192.168.70.135 |
| TEID | 0x00000011 | 0x00000012 | 0x00000013 |

Meaning:

- **Outer IP addresses:** The GTP-U tunnel endpoints.
- **Inner IP addresses:** The original UE/application packet.
- **TEID:** Identifies the GTP-U tunnel at the receiving endpoint.

Take one screenshot showing the selected packet and these fields.
- Scenario A
  ![A Task 6](screenshots/scenarioAtask6.png)

- Scenario B
  ![B Task 6](screenshots/scenarioBtask6.png)

- Scenario C
  ![C Task 6](screenshots/scenarioCtask6.png)

> Some GTP packets are echo or control-related packets. Select one that contains an inner IPv4 packet.

## 9. Task 7 — TCP Retransmissions

Use the **scenario captures** for this task.

The performance traffic is UDP, but iperf3 also uses a small TCP control connection.

Apply this Wireshark filter:

```text
tcp.analysis.retransmission
```

Record the number of displayed packets:

| Scenario | Displayed TCP retransmissions |
| --- | ---: |
| A | |
| B | |
| C | |

> Report the number displayed by Wireshark using the specified filter. Counts may vary with Wireshark analysis settings. Include screenshots showing the filter and displayed count for each scenario. The filter includes any TCP flows that Wireshark can decode in the capture; a displayed retransmission is not, by itself, proof of loss in the UDP test.

**Question:** Do these TCP retransmissions directly represent lost UDP performance packets?

## 10. Task 8 — RTT and Packet Interval

Use a **scenario capture** for both parts of this task.

### TCP RTT

Apply:

```text
tcp.analysis.ack_rtt
```

Select one packet from the iperf3 TCP control connection and record its ACK RTT:

| Scenario | Packet number | Example TCP ACK RTT |
| --- | ---: | --- |
| A, B, or C | | |

This value is the RTT of the TCP control connection. It is not the RTT of the UDP data packets.

### Packet Interval

Apply a display filter to select UDP data packets from the same flow.

Select two consecutive, distinct UDP data packets in the filtered packet list. Avoid duplicated observations caused by merged captures; use the same capture point/interface when possible.

In the packet details of the second packet, find:

```text
Time delta from previous displayed frame
```

Record the scenario, display filter, packet numbers, and time interval.

| Scenario | Display filter | Packet numbers | Time interval |
| --- | --- | --- | --- |
| | | | |

**Note:** The time interval depends on the applied display filter. Make sure the selected packets are distinct data packets, not duplicated observations from different capture points. Do not use their interval to infer the TDD slot pattern.

## 11. Final Questions

Answer each question in two or three sentences:

1. Does Scenario A use a TDD setting that matches its traffic direction?
2. Why is Scenario B expected to have limited uplink resources? Cite a parameter or count from the DL-heavy SIB1 reference configuration.
3. How much does uplink throughput change from B to C?
4. What negative result, if any, is visible in C? Support your answer with packet loss or jitter values.
5. Why can increasing UL resources improve uplink throughput?
6. Is adding more UL resources always the best solution? Explain using throughput and packet loss.
7. Suggest one simple follow-up test that could improve throughput without producing too much packet loss.
8. What can the configuration captures tell you that the iperf3 JSON cannot? Why are separately recorded configuration captures not direct proof of the configuration during an earlier test?

## 12. Submission Checklist

Submit one Markdown file containing:

- the completed direction table;
- the throughput table and B-to-C calculation;
- the B/C packet-loss and jitter table;
- the SIB1 TDD parameter table, including filenames and packet numbers;
- one SIB1 TDD screenshot per configuration, plus `pattern2` screenshots if needed;
- two TDD diagrams: DL-heavy and balanced, with D/U/F labels and mixed-slot symbol details;
- the per-sequence symbol-count / UL-share table, the common 5 ms comparison table, and the configuration-to-performance table;
- the Task 4.5 answers and the TDD suitability table;
- one GTP-U screenshot and its completed table;
- the TCP retransmission counts and screenshots;
- one TCP ACK RTT example;
- one packet-interval example; and
- answers to the final questions.

The PRACH observation is optional and carries no required points. Include all referenced image files alongside the Markdown file, or submit a ZIP containing the report and images.

## 13. Grading Rubric

| Item | Points |
| --- | ---: |
| Traffic direction | 5 |
| Throughput values and calculation | 20 |
| B/C packet-loss and jitter comparison | 15 |
| SIB1 TDD parameter extraction and screenshot evidence | 10 |
| TDD diagrams, symbol counts, and UL shares | 10 |
| TDD suitability and connection to measured performance | 10 |
| GTP-U fields and screenshot | 15 |
| TCP retransmission, RTT, and packet interval | 10 |
| Clear final answers | 5 |
| **Total** | **100** |

## References

These references support the field names and interpretation. Reading the full specifications is not required.

- [Wireshark NR RRC display filter reference](https://www.wireshark.org/docs/dfref/n/nr-rrc.html): SIB1 and TDD display-filter fields.
- [3GPP TS 38.331, Release 18](https://www.etsi.org/deliver/etsi_ts/138300_138399/138331/18.08.00_60/ts_138331v180800p.pdf): SIB1, TDD-UL-DL-ConfigCommon, and TDD-UL-DL-Pattern definitions.
- [3GPP TS 38.213, Release 17, Section 11.1](https://www.etsi.org/deliver/etsi_ts/138200_138299/138213/17.06.00_60/ts_138213v170600p.pdf): Interpretation of common TDD slot and symbol allocation.
