# FIO
Script to run fio, build a graph for clat and histograms for bandwidth and iops
# !/usr/bin/env python3

import subprocess
import json
import matplotlib.pyplot as plt
import numpy as np


def percentile_graph(percentile, latency):
    plt.plot(latency, percentile, color='g')
    plt.ylabel('Percentile')
    plt.xlabel('Latency')
    plt.title('Clat Percentile')
    plt.show()


def hist(iops, name):
    a = np.array(iops)
    plt.hist(a, bins='auto')
    plt.title(name)
    plt.grid(True)
    plt.show()


command = "sudo fio --minimal -name=4k-random-write-libaio  --bs={}k --ioengine=libaio --iodepth=64 --numjobs --size=1G --direct=1 --buffered=0 --per_job_logs=1 --rw=randwrite  --filename=file-$jobnum --numjobs=5 --time_based --runtime=300 --group_reporting --directory=/home/fio/ --output-format=json"
block_sizes = [4, 32, 128]
write_results = list()
for i in block_sizes:
    output = subprocess.Popen(command.format(i), stdout=subprocess.PIPE, shell=True)
    stdout = output.communicate()[0]
    jsonResponse = json.loads(stdout.decode('utf-8'))
    write_dict = jsonResponse.get("jobs")[0].get("write")
    write_results.append(write_dict)
    print("__________________________________________________________________________")
    print(json.dumps(write_dict, indent=2))

bw = list()
iops = list()
for write_dict in write_results:
    bw.append(write_dict.get("bw"))
    iops.append(write_dict.get("iops"))

clat_dict = write_results[0].get("clat").get("percentile")
latency = list(clat_dict.values())
percentile = list(clat_dict.keys())
print("_______________________________________________________")
print("bw " + ' '.join(str(i) for i in bw))
print("iops " + ' '.join(str(i) for i in iops))
print("percentile\n" + ' '.join(str(p) for p in percentile))
print("latency\n" + ' '.join(str(i) for i in latency))

percentile_graph(percentile, latency)
hist(iops, "IOPS")
hist(bw, "BW")
