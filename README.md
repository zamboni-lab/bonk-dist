# BONK (Born–Oppenheimer Non-equilibrium Kinetics)
**`bonk`** is a standalone, zero-dependency executable that takes a precursor compound (**SMILES**, **2D `.mol`/`.sdf`**, or **3D `.sdf`/`.mol`/`.xyz`**) and a list of observed **MS/MS fragment `m/z` values**, and generates ranked candidate **fragment `SMILES` structures**, posterior probabilities (`conf=%`), and a mechanistic explanation of how each fragment ion was formed.

**License & Notice**: (c) Zamboni Lab, ETH Zurich. Provided strictly for **non-commercial academic research and testing only**.


## How It Works: First-Principles Quantum Chemistry

In day-to-day workflow, **`bonk`** addresses the same problem as **ChemFrag**, **MetFrag**, **MS-FINDER**, or **CFM-ID**: given a known or candidate precursor molecule and a list of experimental MS²/MS³ fragment $m/z$ peaks, it assigns chemical structures (`SMILES`) and confidence scores to each peak. However, instead of heuristic rule tables or purely combinatorial graph cutting, `bonk` is built on a **first-principles quantum-chemical and molecular dynamics engine** inspired by (QCxMS)[https://xtb-docs.readthedocs.io/en/latest/qcxms_doc/qcxms.html] and other seminal work by the group of Stefan Grimme (University of Bonn). 

Under the hood, `bonk` embeds a 3D conformer, a GFN2-xTB quantum-chemical engine, Born–Oppenheimer Molecular Dynamics, a RRKM/QET cascade, and much more... and is built for speed.

## Quick Start

No installation, Python environment, or external parameter files are required—everything is self-contained inside `bonk.exe` (Windows) and `bonk` (Linux / WSL2).

```powershell
# Windows (PowerShell / CMD):
.\bonk.exe -c "NC(=O)NCCCC(N)C(=O)O" -m "159.0764, 158.0924, 141.0659, 116.0706, 113.0709, 70.0651"

# Linux / WSL2 (Bash):
chmod +x ./bonk
./bonk -c "NC(=O)NCCCC(N)C(=O)O" -m "159.0764, 158.0924, 141.0659, 116.0706, 113.0709, 70.0651"
```

### Example Output

```text
============================================================================
  BONK! v0.2.0 (c) Zamboni Lab. Non-commercial use & testing only.
============================================================================
  Precursor : NC(=O)NCCCC(N)C(=O)O  (C6H13N3O3, [M+H]+ m/z = 176.10297)
----------------------------------------------------------------------------
query_mz,rank,fragment_smiles,formula,exact_mz,error_mda,error_ppm,confidence_pct,sim_intensity,mechanism_class,comment
159.07640,1,"[NH3+]C(CCCN=C=O)C(=O)O","C6H11N2O3+",159.07642,-0.02,-0.1,74.7,981.3,"Precursor Neutral/Radical Loss","Precursor -NH3: charge-directed deamination (from N1); beta-elimination to ketene/isocyanate (-N=C=O / =C=O)"
159.07640,2,"[NH3+]C(=O)N1CCCC1C(=O)O","C6H11N2O3+",159.07642,-0.02,-0.1,25.3,981.3,"Precursor Neutral/Radical Loss (Regioisomer)","Precursor -NH3 regioisomer: charge-directed deamination (from N9 on C8)"
158.09240,1,"[NH3+]C(=O)N1CCCC(N)C1=O","C6H12N3O2+",158.09240,+0.00,+0.0,59.1,645.2,"Precursor Neutral/Radical Loss","Precursor -H2O: 1,2/1,3 through-space dehydration (from O12); Baldwin 5/6-membered nucleophilic ring closure (lactam/pyrrolone/cyclic iminium)"
116.07060,1,"[NH2+]1C(CCC1)C(=O)O","C5H10NO2+",116.07060,+0.00,+0.0,92.0,812.4,"MD Trajectory Cleavage","MD bond rupture of N4-C5 (+0H); Baldwin 5/6-membered nucleophilic ring closure (lactam/pyrrolone/cyclic iminium)"
70.06510,1,"C1CC=[NH+]C1","C4H8N+",70.06513,-0.03,-0.4,84.2,1000.0,"Secondary Cascade (Cut + Neutral Loss)","Secondary cascade: 1-bond cleavage of C5-N4 (+0H) followed by -HCOOH (formic acid elimination) -> Baldwin 5/6-membered nucleophilic ring closure (lactam/pyrrolone/cyclic iminium)"
```

---

## Command-Line Options

```text
Usage: bonk.exe [OPTIONS] [POSITIONAL]...

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
  -F, --format <FORMAT>      Output format: "csv" or "json" [default: csv] [possible values: csv, json]
  -o, --out <OUT>            Optional output file path (writes to stdout if omitted)
  -h, --help                 Print help
  -V, --version              Print version
```

---

## Usage Examples

### Example 1: SMILES + Fragment `m/z` List (Single Best Structure per Peak, `-k 1`)
Predict the top-ranked structure for the FELIX IRIS-verified fragments of **L-Arginine** (`[M+H]+ = 175.1190`):
```powershell
.\bonk.exe -c "NC(=N)NCCCC(N)C(=O)O" -m "158.0924, 157.1084, 140.0818, 130.0975, 116.0706, 112.0870, 70.0651, 60.0556" -k 1
```

### Example 2: Using a Fragment Peak List File (`-f`)
Run `bonk.exe` on **L-Citrulline** using the included [`examples/citrulline_fragments.txt`](examples/citrulline_fragments.txt) peak list:
```powershell
.\bonk.exe -c "NC(=O)NCCCC(N)C(=O)O" -f examples\citrulline_fragments.txt
```

### Example 3: 2D Structure File (`.mol` / `.sdf`) Input
Run `bonk.exe` on a 2D MDL V2000 `.mol` file ([`examples/aspirin_2d.mol`](examples/aspirin_2d.mol)):
```powershell
.\bonk.exe -c examples\aspirin_2d.mol -m "163.0390, 139.0390, 121.0284, 93.0335, 43.0178"
```

### Example 4: 3D Cartesian Coordinate File (`.xyz` / 3D `.sdf`) Input
Run `bonk.exe` directly on a 3D `.xyz` conformer ([`examples/caffeine_3d.xyz`](examples/caffeine_3d.xyz)):
```powershell
.\bonk.exe -c examples\caffeine_3d.xyz -m "195.0877, 138.0662, 110.0713, 83.0604, 69.0447" -k 1
```

### Example 5: Negative Ion Mode (`--adduct "[M-H]-"`)
Analyze negative-mode `[M-H]-` fragments of **α-Aminoadipic acid**:
```powershell
.\bonk.exe -c "NC(CCCC(=O)O)C(=O)O" --adduct "[M-H]-" -m "142.0504, 116.0712, 98.0606"
```

### Example 6: Export as `.csv` or `.json`
```powershell
# Comma-separated values (.csv) saved to a file:
.\bonk.exe -c "NC(=O)NCCCCC(N)C(=O)O" -m "173.0921, 147.1128, 130.0863, 84.0808" --format csv -o homocitrulline_fragments.csv

# Structured JSON output:
.\bonk.exe -c "O=C(O)C1CCCN1" -m "70.0651" --format json -o proline_fragments.json
```
