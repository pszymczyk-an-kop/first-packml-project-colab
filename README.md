# First PackML Project Colab
Projekt poświęcony przygotowaniu do pracy przy projekcie Stevanato

Poniżej najważniejsze dane wspólne dla każdego

WAŻNE -> Po prezentacji podstawowych struktur i FB przedstawione zostały przykłady tworzenia modułów i unitów

Update: 19.12.2023 - 14:21

## Struktury

### ST_EM
Główna struktura equipment module do komunikacji

Folder: DUTs -> EM:
```ST
// Base structure em
TYPE ST_EM :
STRUCT
	Config :ST_CONFIG;
	Cmd :E_CMD;
	Status :E_Status;
	SequenceStep  :INT;
END_STRUCT
END_TYPE
```

### ST_CONFIG
Struktura konfiguracji

Folder: DUTs -> EM:
```ST
// Structure config
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE ST_CONFIG :
STRUCT
	Functionality :E_Function;
	Data :ST_DATA;
END_STRUCT
END_TYPE
```

### ST_DATA
Struktura danych niskiego poziomu (IN/OUT)

Folder: DUTs -> EM:
```ST
// Structure data contains inputs/outputs/axies
TYPE ST_DATA :
STRUCT
	I :REFERENCE TO ARRAY[1..10] OF BOOL;
	Q :REFERENCE TO ARRAY[1..10] OF BOOL;
	Axis  :ARRAY[1..10] OF AXIS_REF;
	pick_pos :INT ; 					// Q CMD from where girpper picks
	drop_pos :INT ;						// Q CMD to where gripper moves for drop
	act_pos :INT;						// I actual position from of gripper
	give_pos :INT;						// writes value to servo_handle // simulator 
END_STRUCT
END_TYPE
```

### ST_MACHINE
Struktura do przechowywania referencji do FB fbEM

Folder: DUTs -> EM:
```ST
// Utility structure to reference to fbEM
TYPE ST_MACHINE :
STRUCT
	fbEM :REFERENCE TO fbEM;
END_STRUCT
END_TYPE
```

### ST_UN
Struktura do komunikacji UN z EM

Folder: DUTs -> UN:
```ST
// For now empty structure to build master for em-s
TYPE ST_UN :
STRUCT
	Cmd :E_CMD;
	Status :E_Status;
END_STRUCT
END_TYPE
```

### ST_CELL
Struktura do sterowania Cell-em

Folder: DUTs -> Cell:
```ST
TYPE ST_CELL :
STRUCT
	Cmd :E_CMD_CELL;
END_STRUCT
END_TYPE
```

## Enumeratory

### E_Status
Enum statusu dla EM i UN

Folder: DUTs -> EM:
```ST
// Enumerator contains packml state, unfortunatly named status
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE E_Status :
(
	undefined :=0,
	clearing,
	stopped,
	starting,
	idle,
	suspended,
	execute,
	stopping,
	aborting,
	aborted,
	holding,
	held,
	unholding,
	suspending,
	unsuspending,
	resetting,
	completing,
	completed
);
END_TYPE
```

### E_CMD
Enum komend EM i UN

Folder: DUTs -> EM:
```ST
// Enumerator containing em commands
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE E_CMD :
(
	reset :=1,
	start,
	stop,
	hold,
	unhold,
	suspend,
	unsuspend,
	abort,
	clear,
	powerOn
);
END_TYPE
```

### E_CMD
Enum funkcjonalności modułu

Folder: DUTs -> EM:
```ST
// Enumerator containing general function of module
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE E_Function :
(
	None,
	eSlider,
	eSliderWithGripper,
	eXYMover
);
END_TYPE
```

### E_CMD_CELL
Enum komend Cell-a

Folder: DUTs -> Cell:
```ST
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE E_CMD_CELL :
(
	STOP := 0,
	START,
	ABORT
);
END_TYPE
```

### E_pos
Enum pozycji używanych przez FB: fbSliderWithGripperEM

Folder: DUTs -> Cell:
```ST
{attribute 'qualified_only'}
{attribute 'strict'}
TYPE E_pos :
(
	base := 0,
	one,
	two,
	three,
	four
	
);
END_TYPE
```

## Globalne listy zmiennych

### Counters
Konfiguracja liczby poszczególnych modułów w programie

Folder: GVLs:
```ST
{attribute 'qualified_only'}
VAR_GLOBAL CONSTANT
	SliderEMNo :INT := 2;
	SliderWithGripperEMNo :INT := 1;
	XYMoverEMNo :INT := 2;
	
	packmlUnLast :INT := 1;
	packmlEmLast :INT := SliderEMNo + SliderWithGripperEMNo + XYMoverEMNo;
END_VAR
```

### GVL
Konfiguracja globalnych wejść i wyjść, do których są przypisywane we/wy z modułów

Folder: GVLs:
```ST
// Global list with inputs/outputs to control and force variables
{attribute 'qualified_only'}
VAR_GLOBAL
	axis :AXIS_REF;
	aInput :ARRAY[1..10] OF ARRAY[1..10] OF BOOL;
	aOutput :ARRAY[1..10] OF ARRAY[1..10] OF BOOL;
	
	Input :ARRAY[1..10] OF BOOL;
	Output :ARRAY[1..10] OF BOOL;
END_VAR
```

## Podstawowe Funtion Block-i do dziediczenia

### fbEM
Służy jako podstawowy FB od którego tworzy się konkretny moduł, wykorzystywany do kompatybilności między modułami

Folder: POUs -> Module:
```ST
FUNCTION_BLOCK fbEM
VAR_IN_OUT
	em :ST_EM;
	un :ST_UN;
END_VAR

VAR_INPUT
	EnableSim :BOOL;
	CmdSim :BOOL := TRUE;
END_VAR

VAR
	emCmdOld :E_CMD;
END_VAR
```

#### Metoda GetUnCmd()
Służy do odświeżenia aktualnej komendy wystawionej w UN z perspektywy EM

```ST
IF emCmdOld <> un.Cmd THEN
	emCmdOld := un.Cmd;
	em.Cmd := un.Cmd;
END_IF
```

### fbUN
Służy jako podstawowy FB od którego tworzy się konkretny unit

Folder: POUs -> UN:
```ST
FUNCTION_BLOCK fbUN
VAR_IN_OUT
	un :ST_UN;
	cell :ST_CELL;
END_VAR

VAR_INPUT
	EnableSim :BOOL;
END_VAR
```

# Przykłady wykorzystania

## Tworzenie modułu
Przykład modułu z elementami stałymi, przy dziedziczeniu po fbEM moduł ma dostęp do em i un jak również EnableSim. Wymagane jest użycie metody GetUnCmd() na początku fb jak również zdefiniowanie i inicjacja symulatora.

Folder: POUs -> Module
```ST
FUNCTION_BLOCK fbExampleEM EXTENDS fbEM
VAR_INPUT
	// Specyficzne dla modułu zmienne wejściowe
END_VAR

VAR
	// Zmienne lokalne
    fbSignalSim :fbSignalSim;
END_VAR
```

```ST
// Komentarz co do wykorzystanych we/wy
//	I[1] <- Siłownik pozycja 1 (wsunięty) 
//	I[2] <- Siłownik pozycja 2 (wysunięty)
//	I[3] <- Czujnik obecności obiektu (na pozycji 1 (wsunięty)) 
//	Q[1] <- Wysuń siłownik (do pozycji 2)
//	Q[2] <- Wsuń siłownik (do pozycji 1)


GetUnCmd();
fbSignalSim(
	em := em,
	EnableSim := EnableSim,
	CmdSim := CmdSim
);

IF em.Config.Functionality = E_Function.eSlider THEN
    // Praca modułu np. instrukcja warunkowa CASE
END_IF
```

## Tworzenie unitu
Przykład unitu z elementami stałymi, przy dziedziczeniu po fbEM moduł ma dostęp do cell i un jak również EnableSim. Elementy stałe odnoszą się do sekcji automatycznego tworzenia modułów. Nie zaleca się używania wbudowanej metody FB_init do inicjalizacji modułów.

Folder: POUs -> UN
```ST
FUNCTION_BLOCK fbTaskExampleUN EXTENDS fbUN
VAR
	// Arrays with 'em' universal structure and 'fbEM' containing ST_MACHINE structure which contains of references to fbEM function block
	em :ARRAY[1..Counters.packmlEmLast] OF ST_EM;
	fbEM :ARRAY[1..Counters.packmlEmLast] OF ST_MACHINE;
	
	// Array containing all instances fo fbSliderEM with its own iterator
	SliderEM :ARRAY[1..Counters.SliderEMNo] OF fbSliderEM;
	iSliderEM :DINT := 1;
	
	// Array containing all instances fo fbSliderWithGripperEM with its own iterator
	SliderWithGripperEM :ARRAY[1..Counters.SliderWithGripperEMNo] OF fbSliderWithGripperEM;
	iSliderWithGripperEM :DINT := 1;
	
	// Array containing all instances fo fbXYMoverEM with its own iterator
	XYMoverEM :ARRAY[1..Counters.XYMoverEMNo] OF fbXYMoverEM;
	iXYMoverEM :DINT := 1;
	
	i : DINT;
	CaseState :INT := 0;
	declareIterator :DINT;
	moduleWorkIterator :DINT;
	initialConfig :BOOL := TRUE;
END_VAR
```

```ST
// Jednorazowa konfiguracja unitu
IF initialConfig THEN
	initialConfig := FALSE;
	// Wskazanie funkcjonalności modułów dla tablicy em
	em[1].Config.Functionality := E_Function.eXYMover;
	em[2].Config.Functionality := E_Function.eSliderWithGripper;
	em[3].Config.Functionality := E_Function.eXYMover;
	em[4].Config.Functionality := E_Function.eSlider;
	em[5].Config.Functionality := E_Function.eSlider;

	// Pętla FOR do przypisywania tablicy odpowiednich modułów
	FOR declareIterator := 1 TO Counters.packmlEmLast DO
		em[declareIterator].Config.Data.I REF= GVL.aInput[declareIterator];
		em[declareIterator].Config.Data.Q REF= GVL.aOutput[declareIterator];

		CASE em[declareIterator].Config.Functionality OF
			E_Function.eSlider:
				fbEM[declareIterator].fbEM REF= SliderEM[iSliderEM];
				iSliderEM := iSliderEM + 1;
			E_Function.eSliderWithGripper:
				fbEM[declareIterator].fbEM REF= SliderWithGripperEM[iSliderWithGripperEM];
				iSliderWithGripperEM := iSliderWithGripperEM + 1;
			E_Function.eXYMover:
				fbEM[declareIterator].fbEM REF= XYMoverEM[iXYMoverEM];
				iXYMoverEM := iXYMoverEM + 1;
		END_CASE
	END_FOR
END_IF


// Pętla pracy modułów
FOR moduleWorkIterator := 1 TO Counters.packmlEmLast DO
	fbEM[moduleWorkIterator].fbEM(em := em[moduleWorkIterator], un := un, EnableSim := EnableSim, CmdSim := FALSE);
END_FOR

// Reszta programu np instrukcja warunkowa CASE
```