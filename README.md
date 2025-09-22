# SPL-RATORS-Collection-Creation
"Scripts and resources used to generate the SPLØRATORS NFT collection, including image layering, metadata creation, and asset management. Documentation included for setup and usage."

# SPLØRATORS NFT Collection - Creation Process

This repository documents the creation process of the SPLØRATORS NFT collection, from generating images and metadata to preparing for the mint.

## Structure

- `config.js`: Collection configuration, including parameters such as:
  - `width` & `height`: Canvas dimensions for the NFTs.
  - `baseName`: Base name for the NFT editions (e.g., SPLØR-0001).
  - `startEditionFrom` & `editionSize`: Edition numbering and total NFTs to generate.
  - `description` & `symbol`: NFT metadata fields.
  - `seller_fee_basis_points`: Royalties for secondary sales.
  - `baseCreator`: Main creator with their wallet address and share percentage (e.g., 65% base, 35% background).
  - `backgroundCreators`: Optional additional creators for specific backgrounds and their share percentage.
  - `layersOrder`: Order of layers to be composited in each NFT.
  - `neutralAttributes`: Layers that are gender-neutral or have no filtering.
  - `attributeLimits`: Maximum usage per attribute.
  - `outputDir`, `imagesDir`, `metadataDir`: Paths for generated files.
  - `editionPadding`: Number of digits for edition numbering.
  - `genderIdentification`: The NFT generator selects gender randomly (Male/Female) and applies gender-specific attributes (bodies, wings, chains, etc.).
  - `1-of-1 uniqueness`: The generator ensures no duplicate combinations are created, maintaining true 1-of-1 NFTs.
  - `attributesPerGender`: Attributes are filtered and applied according to gender, while neutral attributes are applied to both genders.

- `build.js`: Main script to generate NFT images and metadata, respecting attribute limits, gender filtering, and avoiding duplicate combinations.

- `layers/`: Folder containing assets for each layer (backgrounds, bodies, wings, chains, heads, etc.), organized by gender where applicable.

- `output/`: Folder where the generated NFTs and their JSON metadata files are saved.





- `docs/notes.md`: Notes and documentation of the creative and technical process, including any special considerations or observations.


Repositorio listo para la creación de la colección SPLØRATORS con scripts y documentación.

---

**Estructura completa:**

```
splorators-creation/
├─ .gitignore
├─ README.md
├─ config.js
├─ build.js
├─ layers/              # Your art layers (png/jpg) will be here
│   ├─ Backgrounds/
│   ├─ Body/
│   ├─ Wings/
│   └─ Head/
├─ output/
│   ├─ images/           # NFTs 
│   └─ metadata/         # JSON of each NFT
└─ docs/
    └─ notes.md          # Notes on how the collection was created
```

**.gitignore**

```
node_modules/
output/
*.env
```

# SPLØRATORS NFT Collection - Build Process

Repository for the SPLØRATORS NFT generation workflow, from layer assets to metadata creation.

## Structure

- `config.js` – Collection parameters (layers, attributes, creators, edition size, etc.)
- `build.js` – NFT generation script
- `layers/` – Source assets for each layer
- `output/` – Generated images and metadata
- `docs/notes.md` – Technical notes

## Usage

1. Install dependencies:
```bash
npm install sharp
```

2. Add assets to `layers/` by layer.
3. Run generator.
4. Output images and metadata will appear in `output/`.

**config.js**

```javascript
module.exports = {
  width: 2048,
  height: 2048,
  baseName: "SPLØR",
  startEditionFrom: 1,
  editionSize: 50,
  description: "Each SPLØRATOR is a guardian of cultural time, using art and the digital realm to transform every expression into a legacy that keeps our customs and roots alive.",
  symbol: "SPLOR",
  seller_fee_basis_points: 636,
  baseCreator: { address: "xxxxxxxxxxxxxxxxxxxxxxxxxx", share: 65 },
  backgroundCreators: { /* Backgrounds & shares */ },
  layersOrder: ["Backgrounds", "Body", "Wings", "Chain", "Harness", "Head"],
  neutralAttributes: { "Head": true },
  attributeLimits: { /* Your object with limits */ },
  outputDir: "output",
  imagesDir: "output/images",
  metadataDir: "output/metadata",
  editionPadding: 4
};
```

**build.js**

```javascript
const fs = require("fs");
const path = require("path");
const sharp = require("sharp");
const config = require("./config");

const { width, height, baseName, startEditionFrom, editionSize, description, symbol, seller_fee_basis_points, baseCreator, backgroundCreators, layersOrder, neutralAttributes, attributeLimits, outputDir, imagesDir, metadataDir, editionPadding } = config;

if (!fs.existsSync(outputDir)) fs.mkdirSync(outputDir);
if (!fs.existsSync(imagesDir)) fs.mkdirSync(imagesDir, { recursive: true });
if (!fs.existsSync(metadataDir)) fs.mkdirSync(metadataDir, { recursive: true });

const usedAttributes = {};
const usedCombinations = new Set();

function getLayerElements(layerName) {
  const folder = path.join(__dirname, "layers", layerName);
  if (!fs.existsSync(folder)) return [];
  return fs.readdirSync(folder).filter(f => /\.(png|jpg|jpeg)$/i.test(f)).map(f => ({ name: f, path: path.join(folder, f) }));
}

function selectElement(elements) {
  const available = elements.filter(el => { const limit = attributeLimits[el.name] || Infinity; const used = usedAttributes[el.name] || 0; return used < limit; });
  if (available.length === 0) return null;
  const idx = Math.floor(Math.random() * available.length);
  const selected = available[idx];
  usedAttributes[selected.name] = (usedAttributes[selected.name] || 0) + 1;
  return selected;
}

async function generateEdition(edition) {
  let composite = [];
  let attributes = [];
  let gender = Math.random() < 0.5 ? "Male" : "Female";
  let selectedBackground = null;
  let combinationSignature = [];

  for (const layerName of layersOrder) {
    let elements = getLayerElements(layerName).filter(el => { const limit = attributeLimits[el.name] || Infinity; const used = usedAttributes[el.name] || 0; return used < limit; });
    if ((layerName.toLowerCase().includes("background") || layerName.toLowerCase().includes("body")) && elements.length === 0) {
      const allElements = getLayerElements(layerName);
      if (allElements.length > 0) elements.push(allElements[Math.floor(Math.random() * allElements.length)]);
    }
    if (elements.length === 0) continue;
    const isNeutral = neutralAttributes[layerName] || false;
    let filteredElements;
    if (isNeutral || layerName.toLowerCase().includes("background")) {
      filteredElements = elements;
    } else if (gender === "Male") {
      filteredElements = elements.filter(el => el.name.toLowerCase().startsWith("male_"));
    } else {
      filteredElements = elements.filter(el => el.name.toLowerCase().startsWith("female_"));
    }
    if (filteredElements.length === 0) continue;
    const element = selectElement(filteredElements) || filteredElements[Math.floor(Math.random() * filteredElements.length)];
    if (layerName.toLowerCase().includes("background")) selectedBackground = path.parse(element.name).name;
    composite.push({ input: element.path });
    const attrValue = path.parse(element.name).name;
    if (attrValue && attrValue.toLowerCase() !== "null") { attributes.push({ trait_type: layerName, value: attrValue }); combinationSignature.push(attrValue); }
  }

  const signatureKey = combinationSignature.join("|");
  if (usedCombinations.has(signatureKey)) { console.log("⚠️ Combinación repetida, generando otra..."); return generateEdition(edition); }
  usedCombinations.add(signatureKey);

  let creators = [baseCreator];
  if (selectedBackground && backgroundCreators[selectedBackground]) creators.push(backgroundCreators[selectedBackground]);
  for (let i = creators.length - 1; i > 0; i--) { const j = Math.floor(Math.random() * (i + 1)); [creators[i], creators[j]] = [creators[j], creators[i]]; }

  const padded = String(edition).padStart(editionPadding, "0");
  const fileName = `${baseName}-${padded}.png`;
  const outPath = path.join(imagesDir, fileName);

  await sharp({ create: { width, height, channels: 4, background: { r: 0, g: 0, b: 0, alpha: 0 } } }).composite(composite).png().toFile(outPath);

  const meta = { name: `${baseName}-${padded}`, symbol, description, seller_fee_basis_points, image: fileName, attributes: [{ trait_type: "Gender", value: gender }, ...attributes], properties: { files: [{ uri: fileName, type: "image/png" }], category: "image", creators } };
  const metaPath = path.join(metadataDir, `${baseName}-${padded}.json`);
  fs.writeFileSync(metaPath, JSON.stringify(meta, null, 2));
  console.log(`✅ NFT ${fileName} creado → Género: ${gender}, Creador base: ${baseCreator.address}` + (creators.length > 1 ? ` + Fondo: ${creators[1].address}` : ""));
}

(async () => { console.log("🚀 Generando NFTs con límite de uso y evitando duplicados..."); for (let i = startEditionFrom; i < startEditionFrom + editionSize; i++) await generateEdition(i); console.log("🎉 Listo. Revisa la carpeta:", imagesDir); })();
```

