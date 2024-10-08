---
title: "运维| Linux运维常见脚本"
date: 2024-10-08
lastmod: 2024-10-08
author: 'bdsdc'
linkToMarkdown: true
categories: ['运维']
tags: ['Linux']
---
## rsync拷贝数据
```
#!/bin/bash

if [ $# -ne 1 ]; then
  echo "Error: not enough arguments!"
  echo "Usage is: $0 rsync_src_dir"
  exit 3
fi

# rsync src and dest dir 
r_src_dir=$1
# create date dir 
project_dir="/gri-ziyandata-0"
current_date=$(date +%Y%m%d%H%M) 
volue="/share/ZFS530_DATA"

r_src_dest="$project_dir/${current_date}-tmp"
mkdir -pv  ${r_src_dest}

# start time 
start_time=$(date +%Y-%m-%d\ %H:%M:%S)
echo "Start_time: $start_time"
echo "Rsync task started..."

#data_size=`df -h |grep "share"|grep "extarnel" |grep "/sdb"|awk '{print $2}' `
disk_size=`df |grep "/share/ZFS530_DATA" |awk '{print $2}' `

echo "disk_size: $disk_size"

# rsync data command 
rsync -avP  ${r_src_dir}   ${r_src_dest} | tee -a hg0-rsync-$(date +%Y%m).log  

end_time=$(date +%Y-%m-%d\ %H:%M:%S)
echo "End time: $end_time"
echo "Rsync task completed."
# mv date dir 
if [ $? -eq 0 ];then 
	cd ${project_dir}
	chown 1000.1000  ${current_date}-tmp  
	wait
#	mv ${current_date}-tmp  ${current_date}
fi 
# count size 
transmission_size=$(awk -v val="$disk_size" 'BEGIN {printf "%.2f\n", val / 1024 / 1024}')
# end - start
transmission_time=$(($(date -d "$end_time" +%s) - $(date -d "$start_time" +%s)))
# count g/s

#transmission_rate=$(echo "scale=2; $transmission_size / $transmission_time" | bc) 
transmission_rate=$(awk -v val1=$transmission_size  -v  val2=$transmission_time  'BEGIN {printf "%.2f\n", val1 / val2 }')

printf  "transmission size: %03.2f g\n"    ${transmission_size}
printf  "transmission time: %d s\n"      ${transmission_time}
printf  "transmission rate: %03.2f g/s\n"  ${transmission_rate}

```
