# FreeFileSync / RealTimeSync Extension

**Version:** 1.0.2

This extension integrates FreeFileSync and RealTimeSync for automated file synchronization and backup workflows in DeskStride.

## Features

- **Run FreeFileSync Batch Jobs**: Execute `.ffs_batch` configuration files for one-time or scheduled synchronization
- **Run RealTimeSync Monitoring**: Start continuous folder monitoring with `.ffs_real` configuration files
- **Cross-platform**: Works on Windows, macOS, and Linux
- **Background execution**: Support for both synchronous and asynchronous batch job execution

## Requirements

- FreeFileSync must be installed separately on your system
- Download from: https://freefilesync.org/
- License: FreeFileSync has specific licensing terms - see the official website for details
  - Standard/Donation Editions: Private use only
  - Business Edition: Required for commercial/government use

## Installation

1. Install FreeFileSync on your system
2. Install this extension via Extensions → Install from file
3. Configure the extension settings with paths to your FreeFileSync executables

## Configuration

### Extension Settings

- **FreeFileSync Executable Path**: Path to `FreeFileSync.exe` (or `FreeFileSync` on macOS/Linux)
  - Windows: `C:\Program Files\FreeFileSync\FreeFileSync.exe`
  - macOS: `/Applications/FreeFileSync.app/Contents/MacOS/FreeFileSync`
  - Linux: `/usr/bin/FreeFileSync` or custom install path
- **RealTimeSync Executable Path**: Path to `RealTimeSync.exe` (or `RealTimeSync` on macOS/Linux)
  - Windows: `C:\Program Files\FreeFileSync\RealTimeSync.exe`
  - macOS: `/Applications/FreeFileSync.app/Contents/MacOS/RealTimeSync`
  - Linux: `/usr/bin/RealTimeSync` or custom install path
- **Default Batch File**: Optional default `.ffs_batch` file path

### Creating Batch Jobs

1. Open FreeFileSync application
2. Configure your folder comparison and synchronization settings
3. Go to **Menu → File → Save as batch job...**
4. Save the `.ffs_batch` file to your desired location
5. Use this file path in the **Run FreeFileSync Batch Job** node

### Creating RealTimeSync Configurations

1. Open RealTimeSync application (included with FreeFileSync)
2. Configure your folder monitoring settings
3. Save the `.ffs_real` configuration file
4. Use this file path in the **Run RealTimeSync Monitoring** node

## Nodes

### Run FreeFileSync Batch Job

Executes a FreeFileSync batch job for file synchronization.

**Parameters:**
- **Batch File Path** (required): Path to `.ffs_batch` configuration file
- **Wait for Completion** (optional): Wait for batch job to finish before continuing (default: true)

**Return Codes:**
- `0`: Synchronization completed successfully
- `1`: Synchronization completed with warnings
- `2`: Synchronization completed with errors
- `3`: Synchronization was aborted

**Output:**
- `success`: Boolean indicating if the job completed without errors
- `returnCode`: The exit code from FreeFileSync
- `status`: Human-readable status description
- `batchFile`: Path to the batch file that was executed
- `output`: Standard output from FreeFileSync
- `errors`: Standard error output from FreeFileSync
- `message`: Summary message

### Run RealTimeSync Monitoring

Starts continuous folder monitoring and automatic synchronization.

**Parameters:**
- **RealTimeSync Config File** (required): Path to `.ffs_real` configuration file

**Output:**
- `success`: Boolean indicating if monitoring started successfully
- `status`: "started" if monitoring was initiated
- `configFile`: Path to the configuration file
- `processId`: Process ID of the running RealTimeSync instance
- `message`: Summary message with process information

## Usage Examples

### Scheduled Backup Workflow

1. Create a FreeFileSync batch job for your backup folders
2. Use a scheduler node to trigger the workflow daily
3. Add a **Run FreeFileSync Batch Job** node with your batch file path
4. Add conditional logic based on the return code to handle errors

### Continuous Sync Monitoring

1. Create a RealTimeSync configuration for folders you want to monitor
2. Add a **Run RealTimeSync Monitoring** node to start continuous monitoring
3. The node returns immediately with the process ID
4. Monitoring continues in the background until manually stopped

### Multi-Stage Backup

1. Create multiple batch jobs for different backup stages (e.g., critical files, archives, backups)
2. Chain multiple **Run FreeFileSync Batch Job** nodes in sequence
3. Use conditional logic to handle failures at each stage
4. Add notification nodes to alert on backup completion or errors

## Best Practices

1. **Test batch jobs manually**: Run your batch jobs in FreeFileSync GUI first to verify settings
2. **Handle return codes**: Always check the return code in your workflows to handle errors appropriately
3. **Log output**: Capture and log the output from batch jobs for troubleshooting
4. **Background execution**: Use "Wait for Completion: false" for long-running jobs that don't block your workflow
5. **Path configuration**: Use absolute paths for batch files to avoid working directory issues
6. **Error handling**: Set up proper error handling in FreeFileSync batch jobs (Configure → Handle errors → Stop or Ignore)

## Troubleshooting

### "FreeFileSync not found" error

- Verify FreeFileSync is installed on your system
- Check the extension settings for the correct executable path
- On Windows, ensure the path includes `.exe` extension
- Try leaving the path blank to use system PATH

### Batch job fails with errors

- Run the batch job manually in FreeFileSync GUI to identify the issue
- Check file permissions on source and destination folders
- Verify network paths are accessible
- Review the error output in the node results

### RealTimeSync doesn't start

- Ensure RealTimeSync is included with your FreeFileSync installation
- Check the configuration file path is correct
- Verify folder paths in the configuration exist and are accessible
- Check for existing RealTimeSync processes that might conflict

## Notes

- This extension uses Strategy D (bridge to external process) to avoid licensing complexities with FreeFileSync's GPLv3 + usage restrictions
- Users must comply with FreeFileSync's licensing terms (private use only for standard edition, business license for commercial use)
- The extension does not include FreeFileSync software - it must be installed separately
- Cross-platform compatibility depends on FreeFileSync's support for your operating system
- RealTimeSync runs as a background process and continues until manually stopped or system shutdown

## Contract (v1.0.2)

**Install:** Extensions → **Install from file** → choose `freefilesync-sync.dsext`.

**Permissions:**
- **filesystem**
- **shell**

No example workflow: requires FreeFileSync install.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/freefilesync-sync
python tools/deskstride_ext_cli.py pack extensions/freefilesync-sync -o freefilesync-sync.dsext
```
