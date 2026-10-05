# System Resource Monitoring Extension

**Version:** 1.1.1

This extension provides cross-platform system resource monitoring capabilities using the psutil library. Monitor CPU, memory, disk, and network usage with threshold-based triggers and process management.

## Features

- **CPU Monitoring**: Get CPU usage percentage, core count, frequency, and time distribution
- **Memory Monitoring**: Monitor virtual memory (RAM) and swap space usage
- **Disk Monitoring**: Check disk usage for specific paths or all partitions, includes I/O statistics
- **Process Monitoring**: List running processes with detailed resource usage information
- **Threshold-Based Triggers**: Wait for system resources to reach specific conditions
- **Cross-Platform**: Works on Windows, macOS, and Linux

## Requirements

- Python 3.6+
- psutil library (automatically installed as pip dependency)

## Installation

1. Install this extension via Extensions → Install from file
2. The extension will automatically install psutil as a dependency
3. Configure extension settings for default refresh intervals and monitoring preferences

## Configuration

### Extension Settings

- **Default Refresh Interval (seconds)**: Default interval for resource usage sampling (default: 1 second)
- **Include Per-CPU Usage**: Include individual CPU core usage in CPU monitoring results (default: false)

## Nodes

### Get CPU Usage

Get current CPU usage statistics including percentage, core count, frequency, and time distribution.

**Parameters:**
- **Sampling Interval (seconds)**: Time interval for CPU usage measurement (default: 1.0). Higher values give more accurate readings.
- **Per-CPU Breakdown**: Include individual CPU core usage percentages (default: false).

**Output:**
- `cpuPercent`: Overall CPU usage percentage (or per-core if enabled)
- `cpuCountLogical`: Number of logical CPU cores
- `cpuCountPhysical`: Number of physical CPU cores
- `cpuFrequency`: Current, min, and max CPU frequency (if available)
- `cpuTimes`: CPU time distribution (user, system, idle, nice, iowait, etc.)
- `perCpuPercent`: Array of per-core CPU percentages (if enabled)

**Example Usage:**
```json
{
  "cpuPercent": 45.2,
  "cpuCountLogical": 8,
  "cpuCountPhysical": 4,
  "cpuFrequency": {
    "current": 2400.0,
    "min": 800.0,
    "max": 4800.0
  },
  "cpuTimes": {
    "user": 120.5,
    "system": 45.2,
    "idle": 834.3
  }
}
```

### Get Memory Usage

Get current memory usage statistics including virtual memory (RAM) and swap space usage.

**Parameters:**
- None

**Output:**
- `virtualMemory`: RAM usage statistics (total, available, used, free, percent, etc.)
- `swapMemory`: Swap space usage statistics (total, used, free, percent, I/O)

**Example Usage:**
```json
{
  "virtualMemory": {
    "total": 17179869184,
    "available": 8589934592,
    "used": 8589934592,
    "free": 8589934592,
    "percent": 50.0
  },
  "swapMemory": {
    "total": 4294967296,
    "used": 0,
    "free": 4294967296,
    "percent": 0.0
  }
}
```

### Get Disk Usage

Get current disk usage statistics for a specific path or all partitions. Includes disk I/O statistics when available.

**Parameters:**
- **Path**: Path to check disk usage for. Leave blank for platform default (C:\ on Windows, / on macOS/Linux).
- **All Partitions**: Get usage for all disk partitions instead of a single path (default: false).

**Output:**
- For single path: `path`, `total`, `used`, `free`, `percent`, `diskIo` (I/O statistics)
- For all partitions: `partitions` array with device, mountpoint, fstype, and usage info

**Example Usage:**
```json
{
  "path": "/",
  "total": 500107862016,
  "used": 250053931008,
  "free": 250053931008,
  "percent": 50.0,
  "diskIo": {
    "readBytes": 1073741824,
    "writeBytes": 536870912,
    "readCount": 1024,
    "writeCount": 512
  }
}
```

### List Processes

List running processes with optional filtering and detailed resource usage information.

**Parameters:**
- **Filter by Name**: Filter processes by name (partial match, case-insensitive). Leave blank for all processes.
- **Include Details**: Include CPU usage, memory usage, command line, and other detailed information (default: true).
- **Limit Results**: Maximum number of processes to return. 0 for no limit (default: 0).

**Output:**
- `processes`: Array of process information
- `processCount`: Total number of processes returned
- `filterName`: Applied filter name (if any)

**Example Usage:**
```json
{
  "processes": [
    {
      "pid": 1234,
      "name": "chrome",
      "username": "user",
      "status": "running",
      "cpuPercent": 15.2,
      "memoryUsed": 536870912,
      "memoryPercent": 3.1,
      "createTime": 1695234567.0,
      "cmdline": "/usr/bin/chrome --no-sandbox",
      "exePath": "/usr/bin/chrome"
    }
  ],
  "processCount": 1
}
```

### Wait for Resource Threshold

Wait until a system resource (CPU, memory, or disk) reaches a specified threshold condition. Useful for conditional workflows.

**Parameters:**
- **Resource Type**: Type of system resource to monitor (cpu, memory, disk).
- **Threshold (%)**: Threshold value in percentage (0-100).
- **Condition**: Condition to check against threshold (below, above, equal).
- **Timeout (seconds)**: Maximum time to wait for condition to be met. 0 for no timeout (default: 0).
- **Check Interval (seconds)**: How often to check the resource value (default: 5).

**Output:**
- `success`: Boolean indicating if condition was met
- `status`: "condition_met" or "timeout"
- `resourceType`: Monitored resource type
- `threshold`: Threshold value
- `condition`: Condition that was checked
- `currentValue`: Current resource value when condition met
- `elapsedTime`: Time elapsed in seconds

**Example Usage:**
```json
{
  "success": true,
  "status": "condition_met",
  "resourceType": "cpu",
  "threshold": 80.0,
  "condition": "below",
  "currentValue": 75.5,
  "elapsedTime": 45.2
}
```

## Usage Examples

### Simple CPU Monitor

1. Add a **Get CPU Usage** node
2. Use the output to drive conditional logic
3. Send notifications if CPU usage exceeds threshold

### Automated Cleanup Trigger

1. Add a **Wait for Resource Threshold** node (disk usage above 90%)
2. Add cleanup actions when threshold is met
3. Send confirmation notification after cleanup

### Process Health Check

1. Add a **List Processes** node with filter for critical application
2. Check if process is running and resource usage
3. Restart application if not found or using excessive resources

### System Health Dashboard

1. Add multiple monitoring nodes (CPU, Memory, Disk)
2. Aggregate results in a single workflow
3. Generate periodic health reports

## Best Practices

1. **Sampling Intervals**: Use appropriate sampling intervals for your use case. Shorter intervals (1-2 seconds) for real-time monitoring, longer intervals (5-10 seconds) for periodic checks.
2. **Timeout Handling**: Always set reasonable timeouts for threshold waiting to prevent workflows from hanging indefinitely.
3. **Error Handling**: Process monitoring may fail due to permission issues on some systems. Handle exceptions gracefully.
4. **Resource Limits**: Process listing can be resource-intensive on systems with many processes. Use filters and limits to reduce overhead.
5. **Platform Differences**: Be aware of platform-specific differences in resource monitoring (e.g., CPU frequency may not be available on all systems).

## Troubleshooting

### "psutil not found" error

- Ensure the extension is properly installed
- Check that psutil was installed as a dependency
- Try manually installing: `pip install psutil>=5.9.0`

### Permission denied errors

- Some system monitoring functions may require elevated privileges
- On Linux/macOS, you may need to run with appropriate permissions
- On Windows, some process information may be restricted for system processes

### Inaccurate CPU readings

- Increase the sampling interval for more accurate CPU usage measurements
- CPU usage is calculated over the sampling interval, so very short intervals may be less accurate

### Disk I/O statistics not available

- Disk I/O statistics may not be available on all platforms or filesystems
- This is a limitation of the underlying system, not the extension

## Notes

- This extension uses Strategy A (adopt psutil as pip dependency) - psutil is BSD-3-Clause licensed, which is permissive and suitable for inclusion
- Cross-platform compatibility depends on psutil's support for your operating system
- Some advanced monitoring features may not be available on all platforms due to OS limitations
- The extension provides honest cross-platform parity for system monitoring, complementing the Windows-only WMI extension (backlog item 19)

## Contract (v1.1.1)

**Install:** Extensions → **Install from file** → choose `system-resource-monitor.dsext`.

**Permissions:**
- **filesystem**

See templates/ for the example workflow added on install.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/system-resource-monitor
python tools/deskstride_ext_cli.py pack extensions/system-resource-monitor -o system-resource-monitor.dsext
```
