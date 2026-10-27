this is a command to search for specific text patterns inside files/streams of command outputs. It scans text line by line & prints line that matches your criteria

```
grep [options] "search_pattern" filename
```

Use Cases
1. Search for Text inside a specific file
```
grep "error" server.log
```
2. Case-insensitive search (-i)
```
grep -i "error" server.log
```
3. Search recursively through directories (-r)
```
# To search for a piece of text across all files within a directory and all of its subdirectories
grep -r "TODO" /home/users/projects/
```
4. Display Line Numbers (-n)
```
grep -n "database_connection" config.php
```
5. invert the search / filter out the noise (-v)
```
# shows all lines that do not contain specific pattern (useful for cleaning up heavy log files)
grep -v "debug" system.log
```
6. Pipe output from other commands
```
# check if a specific process (line nginx) is running
ps aux | grep nginx

# check if a specific package is installed
dpkg -l | grep python
```
7. Count the number of matches (-c)
```
grep -c "warning" auth.log
```
8. match whole words only (-w)
```
grep -w "art" document.text
```
9. Print surrounding lines for context (-A -B -C)
- **`-A 3`**: Prints the match and **3 lines after** it.
- **`-B 3`**: Prints the match and **3 lines before** it.
- **`-C 3`**: Prints the match and **3 lines both before and after** it (Context).
```
grep -C 3 "Exception" application.log
```