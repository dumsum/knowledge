[Link](https://team.inria.fr/pacap/software/heptane/)
## HeptaneExtract
- `ConfigExtract.cc`: parse configuration XML, outputs [[readelf]] and [[objdump]] regardless of input method (C, ASM, BINARY). Pass output info to `BuildCfg` in `HeptaneExtract.cc`.
- `HeptaneExtract.cc`: 
	- get sections from readelf
	- basically text parsing
	- creates `ObjdumpSymbolTable` 
```c++
typedef  map<t_address, ListOfString > typeTableFunctions;
class ObjdumpSymbolTable
{
public:
    typeTableFunctions functions;
    typeTableFunctions userFunctions;
    vector<ObjdumpVariable> variables;
    vector<ObjdumpSection> sections;
    t_address gp;
};
``` 

## HeptaneAnalysis