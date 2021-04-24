## 使用Python优雅地实现tail命令

```python
class CmdOutputHandler(IPythonHandler):
    async def post(self):
        data = self.get_json_body()
        cmd = data['cmd']
        proc = await asyncio.create_subprocess_exec(cmd[0], *cmd[1:], stdout=asyncio.subprocess.PIPE)
        await asyncio.wait_for(proc.wait(), timeout=10)
        output = await proc.stdout.read()
        self.finish({'output': output.decode('utf-8')})
```

```python
import errno
import os
 
 
# read at least n lines + 1 more from the end of the file，like /usr/bin/tail
def tail(filename, n, bsize=2048):
    # get newlines type, open in universal mode to find it
    with open(filename, 'rU') as f:
        if not f.readline():
            return
        sep = f.newlines
    assert isinstance(sep, str), 'multiple newline types found, aborting'
    sep_bytes = sep.encode('UTF-8')
 
    line_count = 0
    pos = 0
    with open(filename, 'rb') as f:
        f.seek(0, os.SEEK_END)
 
        while line_count <= n + 1:
            # read at least n lines + 1 more; we need to skip a partial line later on
            try:
                f.seek(-bsize, os.SEEK_CUR)
                line_count += f.read(bsize).count(sep_bytes)
                # after f.read() offset will incr bsize, so we need go back again
                f.seek(-bsize, os.SEEK_CUR)
            except IOError as e:
                if e.errno == errno.EINVAL:
                    bsize = f.tell()
                    f.seek(0, os.SEEK_SET)
                    pos = 0
                    line_count += f.read(bsize).count(sep_bytes)
                    break
                raise e
            pos = f.tell()
 
    def read_last_lines(count=line_count):
        # Re-open in text mode
        with open(filename, 'r') as f:
            f.seek(pos, os.SEEK_SET)  # our file position from above
 
            for line in f:
                # We've located n lines *or more*, so skip if needed
                if count > n:
                    count -= 1
                    continue
                # The rest we yield
                yield line
 
    return ''.join(read_last_lines(line_count))
```
