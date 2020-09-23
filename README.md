<div align="center">

## Loading and calling DLLs at runtime


</div>

### Description

I have seen a few methods of calling DLLs at runtime that have required the programmer to know the name of the DLL and function signature (arguments) before compilation, but this is no good if you want to call a function you know nothing about. For some reason the topic is very poorly documented by Microsoft so, armed with some old C++ code and a couple of beers I set out to make a function that would call an export from a DLL with absolutely no information required at design time. The DLL's file name, the name of the function and the arguments are all supplied at runtime. Supports most integer data types as well as Single and PChar (see article for details).
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[James Mistry](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/james-mistry.md)
**Level**          |Advanced
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |Delphi 5
**Category**       |[Miscellaneous](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/miscellaneous__7-1.md)
**World**          |[Delphi](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/delphi.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/james-mistry-loading-and-calling-dlls-at-runtime__7-775/archive/master.zip)





### Source Code

<html>
<head>
<meta http-equiv="Content-Language" content="en-gb">
<meta http-equiv="Content-Type" content="text/html; charset=windows-1252">
<title>DLL By Name</title>
</head>
<body>
<p><font face="Courier New" size="2"><font color="#008000">{<br>
DLL By Name for Delphi<br>
by James Mistry<br>
Copyright © 2002, James Mistry. All rights reserved.<br>
DLL By Name allows you to access a DLL's function by specifying the DLL<br>
name, the function name and the arguments. DLL By Name handles most data<br>
types including Single and is ideal for creating a DLL-based plugin system<br>
or implementing DLL support in a scripting language.<br>
This was inspired by a C++ version by Adam Straughan.<br>
Unlike C++, however, this function could easily be implemented into a DLL<br>
and accessed from other languages.<br>
You're free to do what you like with the code as long as the original<br>
copyright stays on if you distribute it.<br>
Credit to me is not compulsory in compiled code, but it would be nice - I<br>
don't get much motivation.<br>
You should have seen my victory dance when I got this to work: never has a<br>
fat man moved so fast...<br>
Well, if you think this is complicated I'd advise you stay away from the<br>
VB version unless you enjoy programming in hexadecimal ASM (ughh).<br>
Known Bugs/limitations:<br>
1. In C++, Char and Char* (pointer) can be used. However, only PChar would work
with Char* in Delphi, not<br>
^PChar so I'm left with an equivalent for Char*, but not Char. I'm not a C++
programmer by birth so if anyone else knows<br>
a data type I can use as an equivalent to Char, I'd appreciate an e-mail (jm@sandown-software.com).<br>
2. At the moment the function isn't configured for use with custom types but a
few adaptations (maybe in the next<br>
version) would fix that.<br>
3. The only Real data type presently supported is Single. Because no other Real
types only occupy 4 bytes in memory,<br>
they wouldn't fit into the temporary buffer I assign using a Longword. However,
a couple of small changes should<br>
enable larger types. I'll implement this into version 2.<br>
}<br>
</font><br>
<font color="#0000FF">uses</font><br>
Windows, Dialogs;<br>
<font color="#0000FF">type</font> <font color="#008000">//This is an enumeration
type<br>
</font>ARGTYPE = (ARG_NONE = -1, <font color="#008000">// Terminator or no
arguments<br>
</font>ARG_UI1 = 0, <font color="#008000">// unsigned char</font><br>
ARG_I1, <font color="#008000">// signed char<br>
</font>ARG_UI2, <font color="#008000">// unsigned short<br>
</font>ARG_I2, <font color="#008000">// signed short<br>
</font>ARG_UI4, <font color="#008000">// unsigned long<br>
</font>ARG_I4, <font color="#008000">// signed long<br>
</font>ARG_R4, <font color="#008000">// float<br>
</font>ARG_PUI1, <font color="#008000">// unsigned char* (pointer)<br>
</font>ARG_PI1, <font color="#008000">// signed char* (pointer)<br>
</font>ARG_PUI2, <font color="#008000">// unsigned short* (pointer)<br>
</font>ARG_PI2, <font color="#008000">// signed short* (pointer)<br>
</font>ARG_PUI4, <font color="#008000">// unsigned long* (pointer)<br>
</font>ARG_PI4, <font color="#008000">// signed long* (pointer)<br>
</font>ARG_PR4); <font color="#008000">// float* (pointer)<br>
</font><font color="#0000FF">type</font><br>
DLL_ARG = <font color="#0000FF">record</font> <font color="#008000">//Implements
data types defined in ARGTYPE<br>
</font><font color="#0000FF">case</font> eType: ARGTYPE <font color="#0000FF">of</font>
<font color="#008000">//Here we have the equivalent of absolute addressing in
types (union structs in C++)</font><br>
<font color="#008000">//eType is treated as a member of DLL_ARG and implicitly
declared in the case statement</font><br>
ARG_UI1: (ucVal: PChar); <font color="#008000">//I don't know what data type to
use for this: it's supposed to be the same as a C++ char*<br>
</font>ARG_PUI1: (pucVal: PChar);<br>
ARG_I1: (cVal: ShortInt);<br>
ARG_PI1: (pcVal: ^ShortInt);<br>
ARG_UI2: (usVal: Word);<br>
ARG_PUI2: (pusVal: ^Word);<br>
ARG_I2: (sVal: SmallInt);<br>
ARG_PI2: (psVal: ^SmallInt);<br>
ARG_UI4: (ulVal: Longword);<br>
ARG_PUI4: (pulVal: ^Longword);<br>
ARG_I4: (lVal: Integer);<br>
ARG_PI4: (plVal: ^Integer);<br>
ARG_R4: (fltVal: Single);<br>
ARG_PR4: (pfltVal: ^Single);<br>
<font color="#0000FF">end</font>;<br>
<br>
<font color="#0000FF">implementation</font><br>
<font color="#008000">//************************ Notes ************************<br>
//Reals CAN be returned using the Single type<br>
//which occupies 4 bytes - the same as Longword (unsigned int in C++), the<br>
//data type used for temporary storage in the function call.<br>
//If you want to be able to pass/return anything bigger than Longword<br>
//(e.g. Currency) then you're going to have to re-design the data type<br>
//system I've used.<br>
//It is possible to pass/return custom types but you'd have to<br>
//look into some kind of type-checking system. The memory copying shouldn't<br>
//cause a problem as long as you find the size of any types used. The<br>
//safest way to do this is with SizeOf.<br>
</font><font color="#0000FF">function</font> CallDLLByName(szLibrary, szFunction:
PChar; Arguments: array of DLL_ARG; nArgCount: Integer; var pRetVal: DLL_ARG):
<font color="#0000FF">Integer</font>; <font color="#0000FF">cdecl</font>;
<font color="#008000">//Returns error code<br>
</font><font color="#0000FF">var</font><br>
pFun: Integer; <font color="#008000">//The address of the procedure</font><br>
dwTemp: Longword; <font color="#008000">//Temporary storage - big enough for all
types</font><br>
dwRet: Longword; <font color="#008000">//What to return</font><br>
i: Integer; <font color="#008000">//Iterator for arguments</font><br>
m_hDLL: Integer; <font color="#008000">//The handle to the DLL<br>
</font>Begin<br>
pFun := 0;<br>
dwRet := 0;<br>
m_hDLL := 0;<br>
m_hDLL := LoadLibrary(szLibrary); <font color="#008000">//Load the DLL<br>
</font><font color="#0000FF">if</font> m_hDLL = 0 <font color="#0000FF">then</font>
<font color="#008000">//Load error: DLL either wasn't found or couldn't be
loaded<br>
</font><font color="#0000FF">Begin</font><br>
MessageBox(Form1.Handle, 'DLL load error.', 'Error', 48);<br>
Result := 1; <font color="#008000">//LoadError<br>
</font>exit;<br>
<font color="#0000FF">end</font>;<br>
pFun := Integer(GetProcAddress(m_hDLL, szFunction)); <font color="#008000">
//Cast the return value of GetProcAddress to Integer and assign it to pFun<br>
</font><font color="#0000FF">if</font> pFun <> 0 <font color="#0000FF">then</font>
<font color="#008000">//Procedure address was obtained successfully<br>
</font><font color="#0000FF">Begin</font><br>
<font color="#008000">//Loop through the arguments in reverse</font><br>
<font color="#0000FF">for</font> i := High(Arguments) <font color="#0000FF">
downto</font> Low(Arguments)<font color="#0000FF"> do</font><br>
<font color="#0000FF">Begin</font><br>
<font color="#008000">//Copy data to the temporary buffer</font><br>
CopyMemory(@dwTemp, @Arguments[i].lVal, SizeOf(Longword));<br>
<font color="#008000">//Note that SizeOf can return the amount of memory a
specified data type<br>
//occupies, hence SizeOf(Longword)<br>
//Now put it on the stack (I wish I was a pixie)</font><br>
<font color="#0000FF">asm</font> push dwTemp <font color="#0000FF">end</font>;<br>
end;<br>
<font color="#0000FF">if </font>pRetVal.eType = ARG_R4 <font color="#0000FF">
then</font><br>
<font color="#0000FF">Begin</font><br>
<font color="#0000FF">asm</font><br>
call dword ptr [pFun] <font color="#008000">//Call the function</font><br>
fstp dword ptr [dwRet] <font color="#008000">//Perform a store-and-pop</font><br>
<font color="#0000FF">end</font>;<br>
<font color="#0000FF">end<br>
else<br>
Begin</font><br>
<font color="#0000FF">asm</font> <font color="#008000">//In C++ we could have
just used dwRet = (pFun)() but not in Delphi</font><br>
call dword ptr [pFun] <font color="#008000">//Call the function</font><br>
mov dword ptr [dwRet], eax<br>
<font color="#0000FF">end</font>;<br>
<font color="#0000FF">end</font>;<br>
<font color="#008000">//Unload the stack by looping through Arguments</font><br>
<font color="#0000FF">for</font> i := Low(Arguments) <font color="#0000FF">to</font>
High(Arguments) <font color="#0000FF">do</font><br>
<font color="#0000FF">Begin</font><br>
<font color="#0000FF">asm</font> pop dwTemp <font color="#0000FF">end</font>;<br>
<font color="#008000">//We have to copy the data back because values might have
been<br>
//changed</font><br>
CopyMemory(@Arguments[i].cVal, @dwTemp, SizeOf(Longword));<br>
<font color="#0000FF">end</font>;<br>
<font color="#0000FF">if</font> pRetVal.eType <> ARG_NONE <font color="#0000FF">
then</font> <font color="#008000">//Return the value if requested</font><br>
<font color="#0000FF">Begin</font><br>
pRetVal.eType := ARG_I4; <font color="#008000">//Use the entire buffer</font><br>
pRetVal.lVal := dwRet;<br>
<font color="#0000FF">end</font>;<font color="#0000FF"><br>
end<br>
else<br>
Begin</font><br>
ShowMessage(szLibrary + ' does not export a function called "' + szFunction +
'".');<br>
Result := 2; <font color="#008000">//FunctionError<br>
</font>exit;<br>
<font color="#0000FF">end</font>;<br>
Result := 0;<font color="#008000"> //Success</font><br>
<font color="#0000FF">end</font>;<br>
<font color="#0000FF">procedure</font> Test;<br>
<font color="#0000FF">var</font><br>
GetRet: DLL_ARG;<br>
buf: PChar;<br>
Args: array[0..2] of DLL_ARG;<br>
begin<br>
Args[0].eType := ARG_UI4;<br>
Args[1].eType := ARG_PUI1;<br>
Args[2].eType := ARG_UI4;<br>
Args[0].ulVal := 3912; <font color="#008000">//Replace this with the hWnd of a
window in decimal<br>
</font>Args[2].ulVal := 20; <font color="#008000">//The buffer size for received
text<br>
</font>Args[1].pucVal := '12345678901234567890'; <font color="#008000">
//Populate the buffer<br>
</font>GetRet.eType := ARG_UI4; <font color="#008000">//We want a return value
of type Longword<br>
</font>CallDLLByName('C:\WINDOWS\SYSTEM\user32.dll', 'GetWindowTextA', Args, 3,
GetRet); <font color="#008000">//Call GetWindowTextA</font><br>
ShowMessage(String(Args[1].pucVal)); <font color="#008000">//Show the result<br>
</font><font color="#0000FF">end</font>;<br>
<font color="#0000FF">Begin<br>
<br>
end</font>.</font></p>
</body>
</html>

