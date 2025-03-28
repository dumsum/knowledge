Library providing [[Control Flow Graph|CFG]] objects used by [[Heptane]] 

### Notes
- Can add edge between two CFGs to imitate jump - but semantically troublesome as the existing design assigns ownership to precisely one CFG
	- Manually adding to both CFGs eventually causes double free when destructors called
		- use ref counting pointer
	- Other thought: don't do jumps. Start from C Code.
- No straightforward way to move Nodes from one CFG to another
	- add POP and INSERT functionality