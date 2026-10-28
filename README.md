# bop!
**`bop`** is a standalone executable that takes a precursor compound and a list of observed **MS/MS fragment `m/z` values**, and generates ranked candidate **fragment `SMILES` structures**, posterior probabilities, and a mechanistic explanation of how each fragment ion was formed.

In day-to-day workflow, **`bop`** addresses the same problem as **ChemFrag**, **MetFrag**, **MS-FINDER**, or **CFM-ID**: given a known or candidate precursor molecule and a list of experimental MS²/MS³ fragment $m/z$ peaks, it assigns chemical structures (`SMILES`) and confidence scores to each peak. However, instead of heuristic rule tables or purely combinatorial graph cutting, `bop` is built on a **first-principles quantum-chemical and molecular dynamics engine** inspired by [QCxMS](https://xtb-docs.readthedocs.io/en/latest/qcxms_doc/qcxms.html) and other seminal work by the group of Stefan Grimme (University of Bonn).

Under the hood, `bop` embeds a 3D conformer, a GFN2-xTB quantum-chemical engine, Born–Oppenheimer Molecular Dynamics, a RRKM/QET cascade, and much more... and is built for speed.

**License & Notice**: (c) Zamboni Lab, ETH Zurich. Provided strictly for **non-commercial academic research and testing only**. The public version is capped in speed (100x slower) and throughput (1 spectrum per run)


## Quick Start

Everything is self-contained inside `bop.exe` (Windows), `bop` (Linux / WSL2), and `bop-macos-arm` (macOS Apple Silicon / ARM64).

```powershell
# Windows (PowerShell / CMD):
.\bop.exe -c "NC(=O)NCCCC(N)C(=O)O" -m "159.0764, 158.0924, 141.0659, 116.0706, 113.0709, 70.0651"

# Linux / WSL2 (Bash):
chmod +x ./bop
./bop -c "NC(=O)NCCCC(N)C(=O)O" -m "159.0764, 158.0924, 141.0659, 116.0706, 113.0709, 70.0651"

# macOS Apple Silicon / ARM64 (zsh / bash):
chmod +x ./bop-macos-arm
./bop-macos-arm -c "NC(=O)NCCCC(N)C(=O)O" -m "159.0764, 158.0924, 141.0659, 116.0706, 113.0709, 70.0651"
```

### Example Output

```text
============================================================================
  BOP! v0.3.0 (c) Zamboni Lab. Non-commercial use & testing only.
============================================================================
  Precursor : CC(=O)Oc1ccccc1C(=O)O  (C9H8O4, [M+H]+ m/z = 181.04954)
----------------------------------------------------------------------------

Fragment m/z 180.0499
  [1] [O+]=C(c1ccccc1OC(C)=O)O      # C9H8O4+ | exact m/z=180.04171 (+8.2 mDa, +45.5 ppm, conf=100.0%) | Precursor -H.: homolytic H-atom radical loss

Fragment m/z 163.0390
  [1] [O+]#Cc1ccccc1OC(C)=O         # C9H7O3+ | exact m/z=163.03897 (+0.0 mDa, +0.2 ppm, conf=61.5%) | Precursor -H2O: 1,2/1,3 through-space dehydration (from O13); acylium triple-bond closure (-C#[O+])
  [2] [CH2+][C]Oc1ccccc1C(=O)O      # C9H7O3+ | exact m/z=163.03897 (+0.0 mDa, +0.2 ppm, conf=21.8%) | Precursor -H2O regioisomer: 1,2/1,3 through-space dehydration (from O3 on C2)
  [3] [CH2+]C(=O)Oc1ccccc1[C]O      # C9H7O3+ | exact m/z=163.03897 (+0.0 mDa, +0.2 ppm, conf=16.7%) | Precursor -H2O regioisomer: 1,2/1,3 through-space dehydration (from O12 on C11)

Fragment m/z 138.0311
  [1] [O+]=C(c1ccccc1O)O            # C7H6O3+ | exact m/z=138.03115 (-0.0 mDa, -0.3 ppm, conf=91.8%) | MD bond rupture of C2-O4 (+1H)
  [2] [OH+]=C([C]([CH])[C](O[CH])[CH][CH])O  # C7H6O3+ | exact m/z=138.03115 (-0.0 mDa, -0.3 ppm, conf=8.2%) | 4-bond cleavage breaking [C1-C2, C2-O3, C7-C8, C8-C9] (+2H)

Fragment m/z 121.0284
  [1] [c+]1ccccc1C(=O)O             # C7H5O2+ | exact m/z=121.02841 (-0.0 mDa, -0.0 ppm, conf=60.5%) | MD bond rupture of O4-C5 (+0H)
  [2] [O+]#COc1ccccc1               # C7H5O2+ | exact m/z=121.02841 (-0.0 mDa, -0.1 ppm, conf=24.0%) | Precursor -CH3COOH: McLafferty / 1,2-acetic acid elimination (from C1, C11, O12, O13); acylium triple-bond closure (-C#[O+])
  [3] [O+]#Cc1ccccc1O               # C7H5O2+ | exact m/z=121.02841 (-0.0 mDa, -0.0 ppm, conf=15.6%) | Secondary cascade: 1-bond cleavage of C2-O3 (-1H) followed by -CH2CO (ketene elimination) -> acylium triple-bond closure (-C#[O+])

Fragment m/z 93.0335
  [1] [c+]1ccccc1O                  # C6H5O+ | exact m/z=93.03349 (+0.0 mDa, +0.1 ppm, conf=88.0%) | Secondary cascade: 1-bond cleavage of O4-C2 (+0H) followed by -CO2 (decarboxylation)
  [2] O=C1C=CC=[CH+]C1              # C6H5O+ | exact m/z=93.03349 (+0.0 mDa, +0.1 ppm, conf=7.1%) | Skeletal rearrangement to phenoxy [C6H5O]+ ion
  [3] [CH2+][C]O[C][CH][CH][CH]     # C6H5O+ | exact m/z=93.03349 (+0.0 mDa, +0.1 ppm, conf=4.9%) | 2-bond cross-ring cleavage of C8-C9 and C5-C10 (+1H) followed by -H2O
```

---

## Command-Line Options

```text
Usage: bop.exe [OPTIONS] [POSITIONAL]...

Arguments:
  [POSITIONAL]...  Positional arguments: [COMPOUND] [MZ_1] [MZ_2] ...

Options:
  -c, --compound <COMPOUND>  Compound input: SMILES string OR path to a 2D/3D structure file (.mol, .sdf, .xyz, .smi)
  -m, --mz <MZ>...           Fragment m/z values (comma/space separated) or path to a peak list file
  -f, --mz-file <MZ_FILE>    Optional file containing fragment m/z values (one per line, CSV/TSV, or MSP/MGF peak list)
  -a, --adduct <ADDUCT>      Precursor adduct / ionization mode [default: [M+H]+] (supports [M+H]+, [M-H]-, [M+Na]+, [M]+., etc.)
  -e, --energies <ENERGIES>  CID collision energies in eV or V [default: 14,18,24]
  -n, --ntraj <NTRAJ>        Number of parallel 3D GFN2-xTB Born-Oppenheimer MD trajectories [default: 100]
  -t, --tol <TOL>            Mass matching tolerance in Daltons (Da) [default: 0.05]
  -k, --top-k <TOP_K>        Maximum number of candidate SMILES structures to report per fragment m/z [default: 3]
  -F, --format <FORMAT>      Output format: "default" (screen table), "csv", or "json" [default: default] [possible values: default, csv, json]
  -o, --out <OUT>            Optional output file path (writes to stdout if omitted)
  -h, --help                 Print help
  -V, --version              Print version
```

---

## Usage Examples

### Example 1: SMILES + Fragment `m/z` List (Single Best Structure per Peak, `-k 1`)
Predict the top-ranked structure for the FELIX IRIS-verified fragments of **L-Arginine** (`[M+H]+ = 175.1190`):
```powershell
.\bop.exe -c "NC(=N)NCCCC(N)C(=O)O" -m "158.0924, 157.1084, 140.0818, 130.0975, 116.0706, 112.0870, 70.0651, 60.0556" -k 1
```

### Example 2: Using a Fragment Peak List File (`-f`)
Run `bop.exe` on **L-Citrulline** using the included [`examples/citrulline_fragments.txt`](examples/citrulline_fragments.txt) peak list:
```powershell
.\bop.exe -c "NC(=O)NCCCC(N)C(=O)O" -f examples\citrulline_fragments.txt
```

### Example 3: 2D Structure File (`.mol` / `.sdf`) Input
Run `bop.exe` on a 2D MDL V2000 `.mol` file ([`examples/aspirin_2d.mol`](examples/aspirin_2d.mol)):
```powershell
.\bop.exe -c examples\aspirin_2d.mol -m "163.0390, 139.0390, 121.0284, 93.0335, 43.0178"
```

### Example 4: 3D Cartesian Coordinate File (`.xyz` / 3D `.sdf`) Input
Run `bop.exe` directly on a 3D `.xyz` conformer ([`examples/caffeine_3d.xyz`](examples/caffeine_3d.xyz)):
```powershell
.\bop.exe -c examples\caffeine_3d.xyz -m "195.0877, 138.0662, 110.0713, 83.0604, 69.0447" -k 1
```

### Example 5: Negative Ion Mode (`--adduct "[M-H]-"`)
Analyze negative-mode `[M-H]-` fragments of **α-Aminoadipic acid**:
```powershell
.\bop.exe -c "NC(CCCC(=O)O)C(=O)O" --adduct "[M-H]-" -m "142.0504, 116.0712, 98.0606"
```

### Example 6: Export as `.csv` or `.json`
```powershell
# Comma-separated values (.csv) saved to a file:
.\bop.exe -c "NC(=O)NCCCCC(N)C(=O)O" -m "173.0921, 147.1128, 130.0863, 84.0808" --format csv -o homocitrulline_fragments.csv

# Structured JSON output:
.\bop.exe -c "O=C(O)C1CCCN1" -m "70.0651" --format json -o proline_fragments.json
```

### Example 7: Sodiated `[M+Na]+` Precursors
```powershell
# Sodiated [M+Na]+ precursor with cross-ring & metal-salt eliminations:
.\bop.exe -c "OCC1OC(O)C(O)C(O)C1O" --adduct "[M+Na]+" -m "203.0526, 185.0420, 143.0315, 83.0104"
```