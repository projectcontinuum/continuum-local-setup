# RDKit Test Workflows

This directory contains test workflows for the RDKit Continuum nodes. Each workflow tests a specific node's functionality using the SDFDifferenceChecker node to verify expected outputs.

## Directory Structure

The workflows are organized by phases matching the node categories:

```
test-workflows/rdkit/
├── 01-converters/          # Molecule format conversion nodes
├── 02-modifiers/           # Molecule modification nodes
├── 03-calculators/         # Descriptor and property calculation nodes
├── 04-geometry/            # 3D geometry and conformer nodes
├── 05-fingerprints/        # Fingerprint generation and similarity nodes
├── 06-fragments/           # Molecule fragmentation nodes
├── 07-searching/           # Substructure searching nodes
├── 08-reactions/           # Chemical reaction nodes
├── 09-rendering/           # Molecule rendering nodes
├── 10-experimental/        # Experimental/beta nodes
└── 11-testing/             # Testing and validation nodes
```

## Test Workflow Structure

Each test workflow follows a standard pattern:

1. **Input Data Node** - CreateTableNodeModel providing test molecules as SMILES
2. **Node Under Test** - The RDKit node being tested
3. **Expected Results Node** - CreateTableNodeModel with expected output
4. **Verification Node** - SDFDifferenceCheckerNodeModel comparing actual vs expected

## Workflows by Phase

### Phase 01 - Converters
| Node | Workflow | Description |
|------|----------|-------------|
| CanonicalSmilesNodeModel | test-canonical-smiles.cwf | Tests SMILES canonicalization |
| InChIToMoleculeNodeModel | test-inchi-to-molecule.cwf | Tests InChI to SMILES conversion |
| MoleculeToInChINodeModel | test-molecule-to-inchi.cwf | Tests SMILES to InChI conversion |
| MoleculeWriterNodeModel | test-molecule-writer.cwf | Tests molecule format output |
| SmilesParserNodeModel | test-smiles-parser.cwf | Tests SMILES parsing |

### Phase 02 - Modifiers
| Node | Workflow | Description |
|------|----------|-------------|
| AddHsNodeModel | test-add-hs.cwf | Tests adding explicit hydrogens |
| RemoveHsNodeModel | test-remove-hs.cwf | Tests removing explicit hydrogens |
| AromatizeNodeModel | test-aromatize.cwf | Tests aromaticity perception |
| KekulizeNodeModel | test-kekulize.cwf | Tests Kekule form conversion |
| SaltStripperNodeModel | test-salt-stripper.cwf | Tests salt/counterion removal |

### Phase 03 - Calculators
| Node | Workflow | Description |
|------|----------|-------------|
| DescriptorCalculationNodeModel | test-descriptor-calculation.cwf | Tests molecular descriptor calculation |
| CalculateChargesNodeModel | test-calculate-charges.cwf | Tests Gasteiger charge calculation |

### Phase 04 - Geometry
| Node | Workflow | Description |
|------|----------|-------------|
| AddCoordinatesNodeModel | test-add-coordinates.cwf | Tests 2D/3D coordinate generation |
| AddConformersNodeModel | test-add-conformers.cwf | Tests multiple conformer generation |
| OptimizeGeometryNodeModel | test-optimize-geometry.cwf | Tests force field optimization |

### Phase 05 - Fingerprints
| Node | Workflow | Description |
|------|----------|-------------|
| FingerprintNodeModel | test-fingerprint.cwf | Tests fingerprint generation |
| FingerprintSimilarityNodeModel | test-fingerprint-similarity.cwf | Tests Tanimoto similarity |
| DiversityPickerNodeModel | test-diversity-picker.cwf | Tests MaxMin diversity selection |

### Phase 06 - Fragments
| Node | Workflow | Description |
|------|----------|-------------|
| MurckoScaffoldNodeModel | test-murcko-scaffold.cwf | Tests Murcko scaffold extraction |
| MoleculeExtractorNodeModel | test-molecule-extractor.cwf | Tests molecule fragment extraction |
| MolFragmenterNodeModel | test-mol-fragmenter.cwf | Tests BRICS fragmentation |

### Phase 07 - Searching
| Node | Workflow | Description |
|------|----------|-------------|
| SubstructFilterNodeModel | test-substruct-filter.cwf | Tests substructure filtering |
| MCSNodeModel | test-mcs.cwf | Tests Maximum Common Substructure |

### Phase 08 - Reactions
| Node | Workflow | Description |
|------|----------|-------------|
| OneComponentReactionNodeModel | test-one-component-reaction.cwf | Tests single-reactant reactions |
| TwoComponentReactionNodeModel | test-two-component-reaction.cwf | Tests two-reactant reactions |

### Phase 09 - Rendering
| Node | Workflow | Description |
|------|----------|-------------|
| RDKit2SVGNodeModel | test-rdkit2svg.cwf | Tests SVG rendering |

### Phase 10 - Experimental
| Node | Workflow | Description |
|------|----------|-------------|
| StructureNormalizerNodeModel | test-structure-normalizer.cwf | Tests structure normalization |

### Phase 11 - Testing
| Node | Workflow | Description |
|------|----------|-------------|
| SDFDifferenceCheckerNodeModel | test-sdf-difference-checker.cwf | Tests SDF comparison |

## Running Tests

To run these test workflows:

1. Start the Continuum orchestration service
2. Load a workflow file (`.cwf`) into the Continuum Workbench
3. Execute the workflow
4. Check the SDFDifferenceChecker output for `match_status`:
   - `match` - Test passed, molecules match expected
   - `mismatch` - Test failed, molecules differ
   - `left_only` / `right_only` - Row count mismatch

## Test Data

Test molecules used across workflows include common cheminformatics examples:
- **benzene** (`c1ccccc1`) - Simple aromatic ring
- **ethanol** (`CCO`) - Simple alcohol
- **aspirin** (`CC(=O)Oc1ccccc1C(=O)O`) - Drug molecule with multiple functional groups
- **phenol** (`c1ccc(O)cc1`) - Substituted benzene
- **aniline** (`c1ccc(N)cc1`) - Substituted benzene with nitrogen

## Adding New Tests

To add a new test workflow:

1. Create a new directory under the appropriate phase folder
2. Name the directory after the node model (e.g., `NewNodeModel/`)
3. Create a workflow file with the standard 4-node pattern
4. Include test cases for:
   - Normal operation with valid input
   - Edge cases (empty strings, invalid SMILES)
   - Various configuration options

