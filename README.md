## Phantom V

Phantom V demonstrates powerful transformations available to attackers to avoid hooks, evade analysis, and masquerade as a benign binary. Phantom V is an offensive Portable Executable re-packaging tool in the form of a Python library. When an attack involves executing a Portable Executable file remotely on a target host, the transformations demonstrated through the use of this library demonstrate the potential for malware to evade detection by endpoint security products and sensors.

## Usage

```python
import phantom_v as phantom

sPE = phantom.PE('evil.exe') # source PE | Malware.
tPE = phantom.PE('calc.exe') # target PE | Benign decoy.

sPE.transforms.unhook()
sPE.transforms.obfuscate()
sPE.transforms.evade()
sPE.transforms.masquerade(tPE)
sPE.exports.PE('evil_calc.exe')
```

Each transform has numerous customization options to pass as arguments, see [Advanced Usage](https://github.com/jt0dd/phantom/blob/main/README.md#advanced-usage) for details.

## Defenses

Before outlining the transformations facilitates, it is useful to first understand the defenses they aim to mitigate.

#### **(Defense) Dynamic Analysis:**

Security products commonly perform sandboxed emulation of a executable file. Security products rely on models trained to observe and classify the actions taken during this emulation to include memory artifacts exposed at runtime (visible after decryption or decoding occurs, resource usage patterns, and system call sequences. Observations can be compared either directly or heuristically (a scored, approximated feature similarity approach) for similarity to known-bad samples.

#### **(Defense) Hooking:**

Hooking is a key form of Dynamic Analysis. Security engineers know that emulation is not a reliable medium for analysis. This is where hooking comes into play: Detect the malicious behavior at run-time. So, how does it work? Most programs designed for user-mode execution call out to standard libraries and APIs which are dynamically linked into the program's memory space from shared memory at run-time by the linker. Security products commonly patch hooks into this shared memory space. Hooks work by replacing the entrypoint to an API subroutine with a jump out to an intermediary analytic subroutine before jumping back into the intended API subroutine. Hooking can facilitate analysis of OS API and shared library call sequences and system call sequences.

#### **(Defense) Static Analysis:**

Many traits of an executable file can be useful data sources toward the goal of heuristically or directly scoring similarity to other known-bad samples. Even something as simple as statically measuring entropy in the .text section of an executable can strongly indicate whether some content is encrypted. Shared library imports are apparent through Static Analysis and can be indicators of malicious intent. A file's hash or multiple hashes of file slices can be compared against known-bad samples as part of Static Analysis. 

#### **(Defense) Baselining:**

By performing Static and/or Dynamic Analysis of the processes typically operating on a host or group of hosts within an organization, a baseline of normal activity can be recorded. Data clustering can then be leveraged to allow human or automated analysis to notice, investigate, and react to outliers. 

## Transformations

Phantom V either facilitates the combination of four separate transformations: Unhooking, Evasion, Obfuscation, and Masquerading.

#### **(Counter-Defense) Unhooking:** 
**Mitigates: \[** [_Hooking_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-hooking) **\]**

In order to evade endpoint security hooks within shared libraries, it is necessary to either remove them (an invasive and unstable option) or side-step them by loading a trusted, unhooked clean version of the dependency and store it statically. The Unhooking transformation step mitigates not only Hooking but also Static Analysis. Merely including certain API resources as imports for the linker can tip off a security product to the malicious potential of the program. The process of Unhooking removes such traces from the observation scope of Static Analysis. Unhooking is complicated by one problem: Dynamically linked subroutines exposed by the Windows kernel are wrappers which have different implementations across different OS versions. These cannot be statically linked, unless the attacker wants the responsibility of generating a separate version of the malware for every individual target environment. Fortunately for the attacker, this problem can be side-stepped. Thanks to Windows' Patch Guard, security products cannot apply hooks to kernel components; at least not in modern 64-bit versions of the OS. Therefore, while all other external imports should be consolidated, Windows kernel imports can be left alone.

#### **(Counter-Defense) Evasion:**
**Mitigates: \[** [_Dynamic Analysis_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-dynamic-analysis) **\]**

Emulators have been consistently demonstrated to be riddled with inconsistencies when compared to bare-metal execution. There are too many methods of determining that the runtime is emulated to list here. The user provides machine code which detects the emulator runtime, and the outcome of that machine code is communicated via a zero (no emulator detected) or non-zero (emulator detected) value in the AL register. This will determine whether or not the hidden malicious code will be executed or not.


#### **(Counter-Defense) Masquerading:**
**Mitigates: \[** [_Static Analysis_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-static-analysis), [_Dynamic Analysis_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-dynamic-analysis), [_Baselining_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-baselining) **\]**

In order to mitigate Static Analysis, it is common practice to make the metadata and contents of an executable appear benign. A benign decoy program is used as a target, with the malicious source executable being merged into the decoy with traits of malicious program being masked by those of the benign binary. Masquerading can also address Dynamic Analysis by attempting to imitate the baseline of the defender's environment if the attacker knows or can accurately predict processes normally active within that environment. 

#### **(Counter-Defense) Obfuscation:** 
**Mitigates: \[** [_Static Analysis_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-static-analysis), [_Dynamic Analysis_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-dynamic-analysis),[ _Baselining_](https://github.com/jt0dd/phantom-v/blob/main/README.md#defense-baselining) **\]**

Static and Dynamic Analysis can be inhibited by obfuscating a program's executable data both at rest and at run-time. Obfuscation is, put simply, a way of achieving the same outcome through a different state sequence. Simply encrypting the contents of a program is not a sufficiently effective approach to mitigating Dynamic Analysis. At run-time, those contents must be decrypted for execution and either through emulation or memory inspection they can be analyzed effectively. As a result, it is critical that obfuscation be achieved during every time-slice along the path of execution. If even a single byte-sequence in memory can be matched to a uniquely malicious sequence or heuristically similar sequence, the program can be flagged as suspicious and investigated. By taking into consideration and mirroring behavioral traits a decoy executable which matches the target environment's baseline, it is possible to mitigate Baselining through Obfuscation.

## Advanced Usage

The transforms exposed in Phantom V are all simply abstractions of those functionalities imported from projects I've published separately, since they're all useful as standalone libraries:

- `phantom.transforms.obfuscate` : Mutant, https://github.com/jt0dd/mutant
- `phantom.transforms.unhook` : Introvert, https://github.com/jt0dd/introvert
- `phantom.transforms.evade` : Metal Detector, https://github.com/jt0dd/metal-detector
- `phantom.transforms.masquerade` : Masqueteer, https://github.com/jt0dd/masqueteer

### Obfuscate

### Unhook

### Evade

### Masquerade
