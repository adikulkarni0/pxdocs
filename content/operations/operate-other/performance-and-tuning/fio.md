---
title: FIO Performance
keywords: fio performance, tuning, benchmarking, cos, class of service, production, overhead
description: Check out Portworx performance stats meaured with FIO! Portworx operates typically within less than 3% overhead of the underlying storage hardware.
weight: 200
aliases:
    - /install-with-other/operate-and-maintain/performance-and-tuning/fio/
---
Portworx operates typically within less than 3% overhead of the underlying storage hardware.

Note that {{<companyName>}} recommends the following:

* Minimum resources per server:
  * 4 CPU cores
  * 4 GB RAM
* Recommended resources per server:
  * 12 CPU cores
  * 16 GB RAM
  * 128 GB Storage
  * 10 GB Ethernet NIC

## Examples of Portworx performance

These metrics are recorded by [fio](https://github.com/axboe/fio).

The following graphs show the results of running fio against the underlying baremetal hardware and comparing it to the performance of a Portworx volume that used the underlying hardware for storage provisioning.  The graphs show the overhead, or delta, between running the same test on the raw volume and on a Portworx volume.

In this example, the following Intel server was used:
Intel® Wildcat Pass R2312WTTYS 2U
from PCSD - Product Collaboration Systems Division

* Intel® Wildcat Pass R2312WTTYS 2U
  * 2U rack mountable server
  * 2x Intel® Xeon® processors E5-2650 v3 (25M Cache, 2.30 GHz)
  * 500GB SATA 6Gb/s 7200 RPM 2.5" hard drive
  * 120GB Intel® DC S3500 series (Wolfsville) SAS 6Gb/s 2.5" SSD
  * supports up to 12x 3.5” hot-swap drives and 2x 2.5" hot-swap drives
  * 4x 8GB 2133MHz PC4-17000 ECC RDIMM
  * Matrox G200e (Emulex) On-Board Video
  * It also has an Intel® ethernet controller i350 1Gbe dual-port on-board and IPMI 2.0
* Software
  * Docker version 1.12
  * Centos 7.1
  * {{< pxEnterprise >}} v1.0.8

### Random read performance overhead
![Perf Read](/img/perf-read.png)

### Random write performance overhead
![Perf Write](/img/perf-write.png)

### Benchmarking HowTo

* Step 1: Before installing Portworx, benchmark all the disk drives using hdparm or dd. This is the upper bound of all measurements from here on.
* Step 2: Install Portworx and get a baseline run. Stop Portworx and mount the disk drive(s) on a mountpoint on the host. Run Fio benchmark using this mountpoint as the target.
* Step 3: Verify results. Make sure the IOs are hitting the disk by looking at iostat(1) command. Make sure there are no reads when running write test and vice versa. Compare with results from Step1.
* Step 4: Start Portworx, create a volume. Run Fio benchmark with the same options as the baseline using the attached Portworx device as target.
* Step 5: Verify results, repeat Step3. Also compare with results from Step2.

Note the following Results:
* Total runtime.
* Throughput and IOPS.
* Latency (Completion time per request): 90/95th percentile clat.

Fio Options:

```text
* ioengine: Linux native asynchronous IO engine (libaio).
* blocksize: This is the Block size used for each IO operation and varies from application to application.
* readwrite: IO pattern (read/write sequential/random).
* size: This is the dataset size. Ideally should be greater than the size of host cache to avoid any caching effects.
* direct: Set to true for non-buffered IO. This implies files are opened with O_DIRECT flag which results in IOs bypassing host cache.
* iodepth: This is the number of outstanding/queued IO requests in the system. A higher number typically means greater concurrecy and better resource utilization.
* end_fsync: This causes flushing of dirty data to disk at the end of test. The flush time is included in the measurement.
```

Additional factors to consider when running a write test:
* Do not overwrite. Reusing the same target in repeated tests will end up measuring overwrites instead of appending writes.
* Do not include any verify options. Verification will introduce reads in the test.

Example:
```text
docker run --rm -it -v myvol:/mnt antonipx/fio  --blocksize=64k --filename=/mnt/fio.dat --ioengine=libaio --readwrite=write --size=10G --name=test --direct=1 --iodepth=128 --end_fsync=1
```

Additional Factors to consider when running a read test:
* Every read test should be preceded by a dataset population step.
* Before starting the test make sure the cache is purged, to prevent cache reads.
* Use the option `--readonly` to direct fio to perform read on existing dataset.

Example:
```text
docker run --rm -it -v myvol:/mnt antonipx/fio  --blocksize=64k --filename=/mnt/fio.dat --ioengine=libaio --readwrite=read --size=10G --name=test --direct=1 --iodepth=128 --readonly
```

Note: With basic FIO benchmarking, the goal is to find the max capacity of the system. To achieve that, refrain from introducing any limiting constraints. For example, introducing "fsync" after every write IO is not meant to push system to maximum resource utilization. While it's a valid test, it does not return a correct measurement of the system's sustained IO rate capabilities. Note that this does not mean we are (incorrectly) measuring memory writes. All writes generated by the test must hit the disk before the test completion. The goal is to measure sustained IO rate in a steady state. This means ideally we should exclude ramp-up and teardown time from the measurement interval.
