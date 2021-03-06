We are going to be creating a simple buffer overflow challenge. The first step
is to create an empty directory, let's name it `BufferOverflow1`. Within this
directory, we will place 3 files: our source file (`vuln.c`),
our problem information file (`problem.json`), and our instance generation
file (`challenge.py`).

We'll start with the source file. Copy the following source code into
`vuln.c` and replace the definition of `BUFSIZE` with your desired buffer size.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 64

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
  fflush(stdout);
}

int main(int argc, char **argv){
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  vuln();
  return 0;
}
```

The next thing to create is the `problem.json` file. This will store information
about the problem such as its name, category, score, and so forth. The full
specification for this file can be found [problem-json-spec.md](Problem.json).

Our `problem.json` will look something like:

```json
{
  "name": "Buffer Overflow 1",
  "category": "Binary Exploitation",
  "description": "Exploit the buffer overflow found in {{url_for(\"vuln\")}}. Connect to it with <code>nc {{server}} {{port}}</code>.",
  "score" : 50,
  "hints": [],
  "author": "Your name here",
  "organization": "Example"
}
```

Remember to change the author field to your own name, e.g. "Fred Hacker".

Now we must create out `challenge.py` script that specifies how to create an
instance of our challenge. We must assign this specification to the variable
`Problem`. The API provides a function for creating challenges
that are simply compiled source files, shown below.

```python
from hacksport.problem_templates import CompiledBinary
Problem = CompiledBinary(sources=["vuln.c"], remote=True)
```

It's worth noting the following points:

* The default flag file when using `CompiledBinary` is `flag.txt`. This is
  automatically created with the proper permissions.
* The default compilation options allow an executable stack and disable
  stack canaries. These flags can be set using optional arguments.
* The name of the compiled binary defaults to the source file name with the
  extension removed. Thus, our binary will be named `vuln`, which is what we
  specified in our `problem.json`.

Now that we have all of the files we need, we can try to generate a test
instance. From the directory above `BufferOverflow1`, run
`sudo shell_manager deploy -d BufferOverflow1`.

If there are no errors, this command should output a directory storing the deployed files. Running
`sudo ls -l` on the deployment directory should yield the following:

```
-r--r----- 1 hacksports 1230   32 Jul 20 20:09 flag.txt
-rwxr-sr-x 1 hacksports 1230 7388 Jul 20 20:09 vuln
```

As expected, we have a vulnerable binary named `vuln` that has the setgid
bit on. Thus, when exploited, it will be able to read the flag file.

After confirming that the challenge deploys correctly, we can package it
using `sudo shell_manager package BufferOverflow1`. This should create a `.deb`
file in your current directory that can be distributed and installed
with `sudo dpkg -i example-buffer-overflow-1-1.0-0.deb`.

Continue to the next example problem [here](Cryptography Challenge and Intermediate Usage).