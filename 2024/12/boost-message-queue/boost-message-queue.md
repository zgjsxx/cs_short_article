#!/bin/bash

# 检查是否提供了进程名或PID
if [ -z "$1" ]; then
    echo "Usage: $0 <process_name_or_pid>"
    exit 1
fi

# 进程名或PID
PROCESS_NAME_OR_PID=$1

# 监控的时间间隔（秒）
INTERVAL=1

# 获取进程的 PID
if [[ $PROCESS_NAME_OR_PID =~ ^[0-9]+$ ]]; then
    # 如果输入的是 PID
    PID=$PROCESS_NAME_OR_PID
else
    # 如果输入的是进程名，查找其 PID
    PID=$(pgrep -o $PROCESS_NAME_OR_PID)  # 获取最早启动的进程PID
    if [ -z "$PID" ]; then
        echo "Process '$PROCESS_NAME_OR_PID' not found."
        exit 2
    fi
fi

# 获取进程的峰值虚拟内存占用
get_peak_virtual_memory() {
    local pid=$1
    if [ -f "/proc/$pid/status" ]; then
        awk -F':\t' '/VmPeak/ {print $2}' /proc/$pid/status
    else
        echo "N/A"
    fi
}

# 获取进程的峰值物理内存占用（VmHWM）
get_peak_physical_memory() {
    local pid=$1
    if [ -f "/proc/$pid/status" ]; then
        awk -F':\t' '/VmHWM/ {print $2}' /proc/$pid/status
    else
        echo "N/A"
    fi
}

# 获取进程的虚拟内存占用
get_virtual_memory() {
    local pid=$1
    if [ -f "/proc/$pid/status" ]; then
        awk -F':\t' '/VmSize/ {print $2}' /proc/$pid/status
    else
        echo "N/A"
    fi
}

# 获取进程的物理内存占用（RSS）
get_physical_memory() {
    local pid=$1
    if [ -f "/proc/$pid/status" ]; then
        awk -F':\t' '/VmRSS/ {print $2}' /proc/$pid/status
    else
        echo "N/A"
    fi
}

# 将KB转为MB
convert_kb_to_mb() {
    echo "scale=2; $1 / 1024" | bc
}

# 实时监控函数
monitor_process() {
    local pid=$1
    local peak_virtual_mem=$(get_peak_virtual_memory $pid)
    local peak_physical_mem=$(get_peak_physical_memory $pid)

    local peak_virtual_mem_value
    local peak_physical_mem_value
    local virtual_mem
    local virtual_mem_value
    local physical_mem
    local physical_mem_value

    # 提取峰值虚拟内存和峰值物理内存占用值（去除单位KB）
    peak_virtual_mem_value=$(echo $peak_virtual_mem | awk '{print $1}')
    peak_physical_mem_value=$(echo $peak_physical_mem | awk '{print $1}')

    peak_virtual_mem_value=$(convert_kb_to_mb $peak_virtual_mem_value)  # 转换为MB
    peak_physical_mem_value=$(convert_kb_to_mb $peak_physical_mem_value)  # 转换为MB

    echo "Monitoring process with PID: $pid"
    echo "Peak Virtual Memory Usage (VmPeak): ${peak_virtual_mem_value} MB"
    echo "Peak Physical Memory Usage (VmHWM): ${peak_physical_mem_value} MB"

    # 持续输出进程的 CPU 和内存占用情况
    while true; do
        if [ ! -e /proc/$pid ]; then
            echo "Process $pid has terminated."
            exit 0
        fi

        # 获取进程的 CPU 使用率和实时内存占用 (RSS)
        cpu_mem_usage=$(ps -p $pid -o %cpu,%mem,rss --no-headers)
        if [ -z "$cpu_mem_usage" ]; then
            echo "Process $pid is not running."
            exit 1
        fi

        # 获取实时物理内存占用并转换为 MB
        physical_mem_kb=$(echo $cpu_mem_usage | awk '{print $3}')
        physical_mem_value=$(convert_kb_to_mb $physical_mem_kb)

        # 获取虚拟内存占用并转换为 MB
        virtual_mem=$(get_virtual_memory $pid)
        virtual_mem_value=$(echo $virtual_mem | awk '{print $1}')
        virtual_mem_value=$(convert_kb_to_mb $virtual_mem_value)  # 转换为MB

        # 输出 CPU 使用率，物理内存占用 (RSS)，虚拟内存占用 (VmSize)，以及峰值内存占用
        echo "CPU Usage: $(echo $cpu_mem_usage | awk '{print $1}')% | Physical Memory (RSS): ${physical_mem_value} MB | Virtual Memory (VmSize): ${virtual_mem_value} MB | Peak Virtual Memory (VmPeak): ${peak_virtual_mem_value} MB | Peak Physical Memory (VmHWM): ${peak_physical_mem_value} MB"
        
        sleep $INTERVAL
    done
}

# 启动进程监控
monitor_process $PID
