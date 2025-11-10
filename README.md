# Adobe Premiere Pro .prfpset File Format Analysis

🔬 **Research Project**: Reverse Engineering Adobe Premiere Pro Effect Preset Files

## ⚠️ Legal Disclaimer

This project is for **educational and research purposes only**. 

- Adobe Premiere Pro is a registered trademark of Adobe Inc.
- This analysis is based on publicly accessible .prfpset files
- Reverse engineering for interoperability research purposes
- **NOT for commercial use** - users assume all risks
- No Adobe proprietary code is included or distributed

## 🎯 Project Overview

A comprehensive analysis of Adobe Premiere Pro's `.prfpset` file format, specifically focusing on:

- XML structure of effect presets
- Mask tracking keyframe data format
- Base64-encoded Bezier curve data
- Copy/Paste workflow between effects

## 🔍 Key Discoveries

### XML Structure Analysis
- `.prfpset` files use XML format with specific Adobe schemas
- Mask data stored in `ObjectID="20"` components
- Keyframes stored as child elements (`kf.N.position` + `kf.N.value`)
- Time conversion formula: `position / 254016000000 = seconds`

### Mask Keyframe Format
- Each keyframe: 144-byte Base64-encoded Bezier curve data
- 30 keyframes covering 0.966 seconds of animation
- Binary signature: `3263696e02000000` (hex)
- Contains position, control points, and curve parameters

## 📁 Project Structure

```
├── analysis/
│   ├── prfpset-analysis-report.md     # Complete analysis report
│   ├── sample-files/                  # Sample .prfpset files
│   └── xml-structure/                 # XML schema documentation
├── tools/
│   ├── prfpset-parser.js             # JavaScript parser
│   └── keyframe-extractor.js         # Keyframe data extractor
├── premiere-plugin/
│   ├── mask-track-bridge/            # CEP Plugin source
│   └── extendscript/                 # Premiere Pro scripting
└── docs/
    ├── workflow.md                   # Copy/Paste workflow
    └── file-format.md                # File format specification
```

## 🛠️ Technical Implementation

### Parser Features
- Complete .prfpset XML parsing
- Base64 mask data decoding
- Keyframe time conversion
- Bezier curve coordinate extraction

### Premiere Pro Plugin
- CEP-based extension panel
- Real-time clipboard monitoring
- Mask-to-Transform workflow automation
- Support for Position, Rotation, Scale properties

## 🔬 Research Methodology

1. **File Collection**: Various .prfpset files from different scenarios
2. **XML Analysis**: Manual inspection and automated parsing
3. **Binary Analysis**: Base64 decoding and hex dump analysis
4. **Workflow Testing**: Copy/Paste behavior verification
5. **Cross-Reference**: Comparison with official Adobe behavior

## 📊 Analysis Results

### Supported Effects
- `AE.ADBE Gaussian Blur 2` (Gaussian Blur with mask)
- `AE.ADBE Geometry2` (Transform/Motion effects)
- Mask components with keyframe animation

### Data Format Specifications
- **Time Format**: Adobe internal ticks (254,016,000,000 ticks/second)
- **Mask Data**: 144-byte binary format per keyframe
- **Encoding**: Base64 with checksum verification
- **Keyframe Count**: Variable (typically 30 for 1-second clips)

## 🚀 Usage Examples

### Basic Parser Usage
```javascript
const parser = new PrfpsetParser();
const maskData = parser.extractMaskKeyframes(xmlContent);
console.log(`Found ${maskData.keyframes.length} keyframes`);
```

### Premiere Pro Plugin
1. Load .prfpset file in plugin panel
2. Analyze and display keyframe data
3. Select target properties (Position/Rotation/Scale)
4. Apply to selected clip's Transform effect

## 🔗 Related Projects

- [Adobe CEP Samples](https://github.com/Adobe-CEP/CEP-Resources)
- [Premiere Pro Scripting Guide](https://github.com/docsforadobe/premiere-scripting-guide)

## 📜 License

This research project is released under MIT License for educational purposes.

**Important**: Commercial use may require additional legal review regarding Adobe's intellectual property rights.

## 🤝 Contributing

Contributions welcome for:
- Additional .prfpset file analysis
- Extended effect support
- Workflow improvements
- Documentation enhancements

Please ensure all contributions maintain the educational/research focus.

## 📧 Contact

For research collaboration or questions about the analysis methodology.

---

**Disclaimer**: This is an independent research project. Not affiliated with or endorsed by Adobe Inc.