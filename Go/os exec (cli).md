os/exec lets your Go program start & control other programs/processes on the operating system
- Hey os, run this command for me
```
cmd := exec.Command("ls", "-la")
output, err := cmd.Output()
```
