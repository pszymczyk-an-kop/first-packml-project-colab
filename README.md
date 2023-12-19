# First PackML Project Colab
Projekt poświęcony przygotowaniu do pracy przy projekcie Stevanato

Poniżej najważniejsze dane wspólne dla każdego

## Struktury

### ST_EM
Główna struktura equipment module do komunikacji
```ST
FUNCTION_BLOCK fbEM
VAR_IN_OUT
	em : ST_EM;
	un : ST_UN;
END_VAR

VAR_INPUT
	EnableSim : BOOL;
	CmdSim : BOOL := TRUE;
END_VAR

VAR
	emCmdOld: E_CMD;
END_VAR
```

Metoda GetUnCmd
```ST
IF emCmdOld <> un.Cmd THEN
	emCmdOld := un.Cmd;
	em.Cmd := un.Cmd;
END_IF
```
