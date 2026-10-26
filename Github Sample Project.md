import os
import sys
import argparse
import unittest
from typing import List, Dict, Any

# ==============================================================================
# SECTION 1: Core Business Logic & Exporter
# ==============================================================================

class DataExporter:
    """Handles formatting data and exporting to CSV."""
    
    @staticmethod
    def export_to_csv(data: List[Dict[str, Any]], filepath: str) -> str:
        """Writes a list of dictionaries to a CSV file."""
        if not data:
            raise ValueError("Data list is empty. Nothing to export.")
            
        folder = os.path.dirname(filepath)
        if folder:
            os.makedirs(folder, exist_ok=True)
            
        headers = list(data[0].keys())
        
        with open(filepath, mode='w', encoding='utf-8') as f:
            # Write header
            f.write(",".join(headers) + "\n")
            # Write rows
            for row in data:
                line = [str(row.get(h, "")) for h in headers]
                f.write(",".join(line) + "\n")
                
        return os.path.abspath(filepath)


class MockBusinessScraper:
    """Simulates fetching business lead details."""
    
    def __init__(self, target_category: str, location: str):
        self.category = target_category
        self.location = location

    def fetch_leads(self, count: int = 5) -> List[Dict[str, Any]]:
        """Generates sample lead data for demonstration."""
        leads = []
        domains = ["com", "co.uk", "org", "net"]
        
        for i in range(1, count + 1):
            domain = domains[(i - 1) % len(domains)]
            company_name = f"{self.category.capitalize()} Hub {i}"
            clean_slug = company_name.lower().replace(" ", "")
            
            leads.append({
                "id": str(i),
                "company_name": company_name,
                "category": self.category,
                "location": self.location,
                "email": f"contact@{clean_slug}.{domain}",
                "website": f"https://www.{clean_slug}.{domain}",
                "status": "Verified"
            })
            
        return leads

# ==============================================================================
# SECTION 2: README & Documentation Generator
# ==============================================================================

README_TEXT = """# 🚀 Lead Scout CLI (Single-File Python Edition)

A lightweight command-line tool written in Python to simulate and structure business lead extraction into formatted CSV data files.

## Features
- **Zero External Dependencies**: Uses Python standard libraries only (`argparse`, `unittest`, `os`).
- **Data Export**: Generates cleaned CSV outputs.
- **Built-in Tests**: Includes automated tests inside the main file.

## Usage

### Run Lead Generator CLI
```bash
python lead_scout_project.py --category "Marketing" --location "London" --count 5 --output "leads.csv"
```

### Run Integrated Unit Tests
```bash
python lead_scout_project.py --test
```

## Options
- `-c, --category` : Target industry or niche (default: `Marketing`)
- `-l, --location` : Target location/city (default: `London`)
- `-n, --count`    : Number of records to generate (default: `5`)
- `-o, --output`   : Output CSV path (default: `leads.csv`)
- `-t, --test`     : Run integrated unit tests
"""

def generate_readme():
    """Generates README.md file in the current working directory."""
    with open("README.md", "w", encoding="utf-8") as f:
        f.write(README_TEXT.strip())
    print("📄 Generated 'README.md' for your GitHub repository.")

# ==============================================================================
# SECTION 3: Integrated Unit Tests
# ==============================================================================

class TestLeadScout(unittest.TestCase):
    def test_scraper_output_length(self):
        scraper = MockBusinessScraper("Tech", "Manila")
        leads = scraper.fetch_leads(count=3)
        self.assertEqual(len(leads), 3)

    def test_exporter_creates_file(self):
        data = [{"id": "1", "name": "Acme Inc", "email": "info@acme.com"}]
        test_file = "test_output.csv"
        path = DataExporter.export_to_csv(data, test_file)
        self.assertTrue(os.path.exists(path))
        if os.path.exists(test_file):
            os.remove(test_file)

    def test_export_empty_data_raises_error(self):
        with self.assertRaises(ValueError):
            DataExporter.export_to_csv([], "empty.csv")

# ==============================================================================
# SECTION 4: CLI Entrypoint
# ==============================================================================

def main():
    parser = argparse.ArgumentParser(description="Lead Scout CLI - Business Data Pipeline Tool")
    parser.add_argument("-c", "--category", default="Marketing", help="Business category or niche")
    parser.add_argument("-l", "--location", default="London", help="Target location/city")
    parser.add_argument("-n", "--count", type=int, default=5, help="Number of leads to retrieve")
    parser.add_argument("-o", "--output", default="leads.csv", help="Output CSV path")
    parser.add_argument("-t", "--test", action="store_true", help="Run internal suite of unit tests")

    args = parser.parse_args()

    if args.test:
        print("🧪 Running internal unit test suite...\n")
        suite = unittest.TestLoader().loadTestsFromTestCase(TestLeadScout)
        runner = unittest.TextTestRunner(verbosity=2)
        result = runner.run(suite)
        sys.exit(0 if result.wasSuccessful() else 1)

    print(f"🔍 Searching for '{args.category}' businesses in '{args.location}'...")
    
    try:
        scraper = MockBusinessScraper(target_category=args.category, location=args.location)
        results = scraper.fetch_leads(count=args.count)
        
        filepath = DataExporter.export_to_csv(results, args.output)
        generate_readme()
        
        print(f"✅ Successfully exported {len(results)} leads to: {filepath}")
    except Exception as e:
        print(f"❌ Error occurred: {str(e)}")

if __name__ == "__main__":
    main()