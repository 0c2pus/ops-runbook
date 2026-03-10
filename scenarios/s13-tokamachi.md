# Scenario: Tokamachi - Troubleshooting a Named Pipe

## 🚩 Issue

A process reads from the named pipe `/home/admin/namedpipe`. When a writer sends
messages in a tight loop, the reader works for a short time and then stops
receiving messages or slows down significantly.

## 🔍 Investigation

### Step 1: Create the named pipe
```bash
mkfifo namedpipe
```

_This creates a special FIFO file. The `p` file type in `ls -la` output confirms
it is a pipe, not a regular file. Data written to it is never stored on disk —
it exists only during transmission between two processes._

### Step 2: Start the reader
```bash
./start-read.sh
```

### Step 3: Run the original writer and observe the failure
```bash
/bin/bash -c 'while true; do echo "this is a test message being sent to the pipe" > /home/admin/namedpipe; done' &
tail -f reader.log
```

_Observation: After ~1 minute the reader stops receiving messages. The writer
loop blocks because the pipe buffer is full — the reader cannot consume messages
fast enough when the writer sends without any delay._

## ❌ What Didn't Work

- Tight loop without delay — writer fills the pipe buffer faster than the reader
  can drain it, causing the writer to block and the entire pipeline to stall.

## ✅ Root Cause

A named pipe has a fixed kernel buffer. When the writer produces messages faster
than the reader consumes them, the buffer fills up and the writer blocks,
waiting for space to free up. This causes the apparent "freeze" in the reader
log.

## 🛠 Resolution

Added a `sleep 2` delay to the writer loop, giving the reader enough time to
consume each message and free buffer space before the next write:
```bash
/bin/bash -c 'while true; do echo "this is a test message being sent to the pipe" > /home/admin/namedpipe; sleep 2; done' &
```

Verified the fix:
```bash
tail -f reader.log
```

_Messages continued to appear consistently without interruption._

## 💡 Lessons Learned

- A named pipe (FIFO) is not a file on disk — data exists only during the
  transfer between two processes and is gone once the reader consumes it.
- Named pipes have a fixed kernel buffer. If the writer is faster than the
  reader, the writer blocks — it does not crash or drop data.
- In a tight loop writing to a pipe, always consider the reader's consumption
  speed. A small `sleep` is often enough to prevent buffer saturation.