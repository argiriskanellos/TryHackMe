
# Corridor Walkthrough

Upon visiting the site on port 80, we find 13 doors. Clicking on each door takes us to a different page with a URL structure like this:

```
http://TARGET_IP/<String_of_number>
```

The first step is to check if the string is a hash. After examining, we determine that each string is encrypted with MD5. After checking all the strings, we observe that each hash corresponds to a number from 1 to 13.

Since our goal is to exit the room, we will try to find the MD5 hash of 0, which is:

```
cfcd208495d565ef66e7dff9f98764da
```

By navigating to the page:

```
http://TARGET_IP/cfcd208495d565ef66e7dff9f98764da
```

We find the FLAG, which is:

```
flag{2______________________________e}
```
