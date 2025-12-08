# Mapping the Market: Uncovering Brand Alliances from Consumer Cross-Shopping Networks


## Overview

This capstone project analyzes customer cross-shopping patterns in San Diego County to identify business communities and generate partnership recommendations. Using network analysis and community detection algorithms, the project reveals latent relationships between businesses based on shared customer behavior, enabling data-driven strategic partnerships.

## AI Usage

We utilized AI (ChatGPT, Github Copilot) to help strategize how to model our business data as a graph.
We also used AI to help structure this README file.
Code completion and style formatting was also AI assisted.


## Project Structure

```
ADS599_Capstone_Project/
├── code/                         # Notebook directory
|   ├── data-prep.ipynb              # Data acquisition and preprocessing
|   ├── exploratory_analysis.ipynb   # Exploratory data analysis and visualizations
|   └── modeling.ipynb               # Network construction, community detection, and recommendations
├── data/                         # Data directory (not in repo)
│   ├── san-diego-county-places.parquet
│   ├── san-diego-county-spend-patterns.parquet
│   └── san-diego-county-places-spend.parquet
├── maps/                         # Example map outputs
│   ├── encinitas_map_check.html
|   └── mcdonalds_2mile_communities.html
|── README.md
└── requirements.txt              # Python dependencies
```

## Data Sources

The project uses **SafeGraph** data from July 2025:

1. **Global Places Dataset**: Point-of-interest (POI) data including:
   - Business locations (latitude/longitude, addresses, postal codes)
   - Business categories (NAICS codes, top categories)
   - Brand affiliations
   - Geographic polygons

2. **Spend Patterns Dataset**: Transaction-level insights including:
   - Customer and transaction counts
   - Total spend metrics
   - Cross-shopping patterns (physical brands, online merchants, local brands, same-category brands)
   - Customer demographics and behavior metrics

**Geographic Scope**: San Diego County, California (all zip codes)
**Time Period**: July 2025 (cross-sectional analysis)
**Final Dataset**: 8,537 business locations after filtering and joining

## Methodology

### 1. Data Preparation (`data-prep.ipynb`)

- Filters SafeGraph data to San Diego County zip codes
- Joins Global Places and Spend Patterns datasets on `PLACEKEY`
- Exports cleaned, joined dataset to Parquet format for efficient processing
- Handles duplicate columns and missing values

**Key Steps**:
1. Load Global Places data filtered by San Diego County zip codes, downloaded via the Dewey platform
2. Load Spend Patterns data with brand information 
3. Join datasets on `PLACEKEY` identifier
4. Export final dataset: `san-diego-county-places-spend.parquet`

### 2. Exploratory Data Analysis (`exploratory_analysis.ipynb`)

Comprehensive analysis of the dataset including:

- **Data Quality Assessment**: Missing values, data types, distributions
- **Business Categories**: Distribution across NAICS categories
- **Geographic Distribution**: Spatial patterns across San Diego County
- **Spending Metrics**: Customer counts, transaction volumes, total spend
- **Cross-Shopping Patterns**: Analysis of related brand shopping behaviors
- **Demographic Insights**: Customer income and frequency patterns

**Key Findings**:
- 8,537 business locations across San Diego County
- Diverse business categories with restaurants and retail dominating
- Cross-shopping percentages stored as JSON strings and requiring processing

### 3. Network Modeling (`modeling.ipynb`)

#### 3.1 Network Construction

**Graph Structure**:
- **Nodes**: Business locations (identified by location name and coordinates)
- **Edges**: Cross-shopping relationships (weighted by percentage)
- **Edge Weight Threshold**: 25% minimum cross-shopping percentage (reduces noise)

**Implementation Details**:
- Parses JSON-formatted cross-shopping data (`RELATED_CROSS_SHOPPING_PHYSICAL_BRANDS_PCT`)
- Creates directed graph where edges represent customer flow between businesses
- Uses location name as brand identifier
- Aggregates edge weights when multiple locations share the same brand

**Final Graph Statistics**:
- **Nodes**: 8,523 (after removing isolated nodes)
- **Edges**: 6,092,856
- **Global Clustering Coefficient**: 0.593 (high clustering indicates strong community structure)

#### 3.2 Community Detection

Two algorithms are implemented and compared:

**Louvain Algorithm**:
- Fast greedy modularity maximization
- Hierarchical community aggregation
- **Modularity Score**: 0.133
- **Communities Detected**: 3 major communities

**Leiden Algorithm**:
- Improved version of Louvain with guaranteed connectivity
- Better for real-world networks with disconnected subgroups
- **Modularity Score**: 0.097
- More stable and interpretable communities, but low modularity output on global graph

**Evaluation Metrics**:
- Modularity: Measures quality of community partition
- Global Clustering Coefficient: Indicates network structure
- Community size distribution

#### 3.3 Partnership Recommendations

A scoring system identifies optimal business partnerships:

**Partnership Score Components** (weighted):
1. **Cross-Shopping Weight (40%)**: Percentage of customers who shop at both businesses
2. **Bidirectional Relationship (20%)**: Mutual cross-shopping (both directions)
3. **Customer Similarity (15%)**: Similarity in customer base sizes
4. **Geographic Proximity (15%)**: Distance-based score (decay over 10 miles)
5. **Community Membership (10%)**: Same community bonus

**Recommendation Function**:
- Filters candidates within specified radius (default: 2 miles)
- Scores all first-degree neighbors
- Returns top N recommendations with detailed score breakdowns
- Generates interactive maps showing recommended partnerships

**Total Score Formula**:
```
Total Score = 0.4 × CSW + 0.2 × B + 0.15 × Sim + 0.15 × Prox + 0.1 × Comm
```

Where:
- CSW = Cross-shopping weight
- B = Bidirectional relationship
- Sim = Customer similarity
- Prox = Proximity score
- Comm = Same community bonus

#### 3.4 Visualization

**Interactive Maps** (Folium):
- Community visualization with color-coded business groups
- Partnership recommendations highlighted
- Geographic filtering and radius-based analysis
- Popup information: business details, recommendation scores, metrics

**Example Outputs**:
- `mcdonalds_2mile_communities.html`: Community map for McDonald's location
- `encintas_map_check.html`: Geographic validation map

## Key Results

1. **Network Structure**: High clustering coefficient (0.593) indicates strong community structure in San Diego County business network

2. **Community Detection**: 
   - Louvain algorithm identified 3 major communities with modularity of 0.133
   - Leiden algorithm provides more granular, well-connected communities

3. **Partnership Opportunities**: The recommendation system successfully identifies businesses with:
   - High customer overlap
   - Mutual cross-shopping relationships
   - Geographic proximity
   - Similar customer demographics

4. **Business Insights**: Communities reveal patterns beyond traditional business categories, showing how customer behavior creates natural business groupings

## Technical Stack

### Core Libraries
- **pandas**: Data manipulation and analysis
- **numpy**: Numerical computations
- **duckdb**: Fast SQL queries on Parquet files
- **networkx**: Graph construction and analysis
- **igraph**: Advanced graph algorithms
- **leidenalg**: Leiden community detection
- **geopandas**: Geospatial data handling
- **folium**: Interactive map generation

### Data Processing
- **pyarrow**: Parquet file I/O
- **json**: Cross-shopping data parsing
- **shapely**: Geometric operations

### Visualization
- **matplotlib**: Statistical plots
- **folium**: Interactive maps
- **pyvis**: Network visualizations

See `requirements.txt` for complete dependency list.

## Usage

### Prerequisites

1. **SafeGraph Data Access**: 
   - Requires SafeGraph API access via Dewey
   - Download Global Places and Spend Patterns datasets for California
   - See `data-prep.ipynb` for download instructions

2. **Python Environment**:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

### Running the Analysis

1. **Data Preparation**:
   ```bash
   jupyter notebook data-prep.ipynb
   ```
   - Update data paths in the notebook
   - Run all cells to generate `data/san-diego-county-places-spend.parquet`

2. **Exploratory Analysis**:
   ```bash
   jupyter notebook exploratory_analysis.ipynb
   ```
   - Review data quality and distributions
   - Generate visualizations and summary statistics

3. **Modeling**:
   ```bash
   jupyter notebook modeling.ipynb
   ```
   - Construct network graph
   - Run community detection algorithms
   - Generate partnership recommendations
   - Create interactive visualizations

### Example: Generate Partnership Recommendations

```python
# Load the graph (from modeling.ipynb)
import pickle
with open('clustering_results.pkl', 'rb') as f:
    graph_data = pickle.load(f)

G = graph_data['graph']
communities = graph_data['leiden_communities']

# Get recommendations for a specific business
business_node = "McDonald's (32.545093, -117.038717)"
recommendations = recommend_partnerships(
    G, 
    business_node, 
    communities=communities,
    top_n=10,
    radius_mi=2
)

# Visualize on map
create_community_map(
    G,
    communities,
    center_node=business_node,
    recommendations=recommendations,
    output_file='recommendations_map.html'
)
```

## Output Files

- `data/san-diego-county-places-spend.parquet`: Final joined dataset
- `clustering_results.pkl`: Saved graph and community detection results
- `clustering_results.csv/parquet`: Clustering coefficient metrics
- `*.html`: Interactive maps (community visualizations, recommendations)
- `graphs/*.gexf`: Graph files for Gephi visualization (if exported)

The notebooks in this repository provide the complete implementation and reproducible analysis supporting the research findings.

## Key Insights for Business Applications

1. **Strategic Partnerships**: Identify businesses with complementary customer bases for cross-promotion and co-location strategies

2. **Market Segmentation**: Community detection reveals natural market segments based on customer behavior rather than traditional categories

3. **Location Intelligence**: Geographic analysis helps identify optimal locations for new businesses based on existing community structures

4. **Customer Behavior**: Cross-shopping patterns reveal how customers move between businesses, informing marketing and placement strategies

## Limitations

1. **Temporal Scope**: Analysis uses cross-sectional data (July 2025); longitudinal analysis could reveal seasonal patterns

2. **Data Coverage**: SafeGraph data may not capture all businesses or customer segments equally

3. **Edge Threshold**: 25% cross-shopping threshold may exclude some valid but weaker relationships. This is tested but analysis can be expanded. 

4. **Geographic Scope**: Limited to San Diego County; results may not generalize to other regions


## Future Work

1. **Temporal Analysis**: Incorporate time-series data to track community evolution
2. **Multi-dimensional Networks**: Extend to online merchants, local brands, and category-based networks
3. **Machine Learning**: Train models to predict partnership success based on network features
4. **Validation**: Partner with businesses to validate recommendation effectiveness
5. **Scalability**: Optimize algorithms for larger geographic regions or national analysis


## License

See `LICENSE` file for details.

## Contact

For questions or collaboration opportunities, please contact:
- **Author**: Christian Lee & Nolan Peters
- **Institution**: University of San Diego, MSADS Program
- **Course**: ADS-599 Capstone Project

---