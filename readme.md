# Chillhop ElasticSearch PHP library

This library is meant to be used as a base for ElasticSearch connections in Chillhop Node/TS applications.

The ChillhopElasticDatabase class needs to be extended to be used

## Usage

Include this library by adding this repository in composer.json

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/ChillhopMusic/chillhop-elasticsearch-php"
        }
    ],
    "require": {
        "php": ">=8.0",
        "chillhopmusic/chillhop-elasticsearch-php": "dev-main"
    },
}
```

In this simple example, an elasticDatabase is created using monthly indexes and an item is indexed into the DB

```php
use ChillhopMusic\ElasticSearch\ChillhopElasticDatabase;

class Test_Record {
    public function __construct(
        public string $id,
        public string $name,
        public string $date,
        public int $amount,
    ) {
    }
}

const TEST_RECORD_MAPPINGS = [
    'name' => ['type' => 'keyword'],
    'date' => ['type' => 'date'],
    'amount' => ['type' => 'integer'],
];

class Test_Database extends ChillhopElasticDatabase {
    protected function createItemId(object $item): string | null {
        // Returning null here would create an item id automatically
        return $item->id;
    }

    public function getItemDate(object $elasticItem): DateTime|null {
        // Convert item date into DateTime, this gets used for assigning the right index
        return new DateTime($elasticItem->date);
    }

    public function getIndexNameWithPrefix(string $indexPrefix, DateTime $utcDate): string {
        // Create a monthly index in the format of index_prefix_YYYY-MM
        return $indexPrefix . $utcDate->format("Y-m");
    }
}

// Create instance
$test_database = new Test_Database(
    null, // If no client is provided, localhost:9200 will be used
    "shortlink_clicks_", // Index prefix
    TEST_RECORD_MAPPINGS
);

// Create a test item
$test_item = new Test_Record(
    "item20200101123",
    "Test Record",
    "2020-01-01",
    5
);

// Index the item
$test_database->index($test_item);

// Check if record exists
$item_exists = $test_database->recordExists($test_item->id, new DateTime("2020-01-01"), null);

echo "Item exists: " . ($item_exists ? "YES" : "NO") . "\n";
```
