# Machine Learning Therapist Matcher

A machine learning-based external service task for Camunda BPM that uses logistic regression to match patients with therapists based on their preferences and availability.

## Overview

This Deepnote notebook implements an cleint-therapist matching system that:
- Trains a logistic regression model on patient-therapist compatibility data
- Maintains a database of 20 mock therapists with various specialties and availabilities
- Integrates with Camunda BPM as an external service task worker
- Automatically evaluates and ranks therapists based on match probability
- Returns the top 3 matches with confidence scores

## Features

- **ML-Powered Matching**: Logistic regression classifier
- **Multi-Criteria Evaluation**: Matches based on 5 key factors:
  - Therapy type (Individual/Couple)
  - Modality preference (Online/Face-to-Face)
  - Day availability
  - Time slot (Morning/Afternoon)
  - Gender preference
- **Camunda Integration**: External service task worker that polls for matching requests
- **Top-N Recommendations**: Returns ranked list of therapists with match probability scores
- **Robust Error Handling**: Comprehensive logging and exception management
- **Real-time Processing**: Continuous polling for new matching tasks

## Technology Stack

- **Python 3.11**
- **scikit-learn**: Logistic regression model
- **pandas & numpy**: Data manipulation
- **joblib**: Model serialization
- **requests**: HTTP client for Camunda REST API
- **cam**: Camunda external task client library: Transparency-note: Author of the cam.py-file in the notebook is Prof. Andreas Martin as to be seen in the headline

## Setup

This is a Deepnote notebook. To run the ML matcher:

1. **Open the notebook in Deepnote:**
   
   [Open ML Therapist Matcher Notebook](https://deepnote.com/workspace/DHP25-244a274b-59d3-442f-b7ef-3d5d24503cee/project/chocolatexpress-fbdcce36-fd51-4cfb-8676-e6e544158098/notebook/Logistic-Classification-Machine-Learning-Matcher-738d2ebcd5ce4bdba232b837d817c7f9?secondary-sidebar-autoopen=true&secondary-sidebar=agent)

2. Configure your Camunda connection:
   - Set `tenant_id` (default: `'mi25chocolate'`)
   - Set `camunda_rest_endpoint` (default: `'https://digibp.engine.martinlab.science/engine-rest'`)

3. Run all cells sequentially:
   - Cell 1: Train the logistic regression model
   - Cell 2: Generate mock therapist data
   - Cell 3: Test Camunda connectivity
   - Cell 4: Start the external task worker

4. The worker will continuously poll for tasks on the `therapistMatch` topic

All dependencies are pre-installed in the Deepnote environment.

## Architecture

### 1. Model Training

The system generates synthetic training data with 5,000 samples, where each sample represents a patient-therapist pairing with binary features:

```python
Features (5 binary values):
- therapy_type_match: 1 if types match, 0 otherwise
- modality_match: 1 if modalities match, 0 otherwise  
- day_match: 1 if days match, 0 otherwise
- time_slot_match: 1 if time slots match, 0 otherwise
- gender_match: 1 if gender preference matches, 0 otherwise

Label: 1 if sum(features) >= 4 (good match), 0 otherwise
```

**Model Performance:**
- Training Accuracy: 100%
- Test Accuracy: 100%
- Class weights: Balanced

### 2. Therapist Database

Mock database of 20 therapists with attributes:
- **therapist_id**: Unique identifier (1-20)
- **name**: Therapist name (e.g., Dr. Smith, Dr. Lee)
- **therapy_type**: Individual or Couple therapy
- **modality**: Online or Face-to-Face
- **days_available**: Monday through Friday
- **time_slot**: Morning (8:00) or Afternoon (14:00)
- **gender**: Male or Female

### 3. Matching Process

For each patient request:

1. **Feature Extraction**: Calculate binary features comparing patient preferences with each therapist's profile
2. **Probability Prediction**: Use trained model to predict match probability for all 20 therapists
3. **Ranking**: Sort therapists by match probability (descending)
4. **Selection**: Return top 3 matches with scores

## Camunda Integration

### External Task Configuration

**Topic Name:** `therapistMatch`

### Input Variables (from Camunda Process)

The worker expects these variables from the BPMN process:

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `therapyType` | String | Type of therapy needed | "Individual" or "Couple" |
| `modality` | String | Preferred therapy format | "Online" or "Face to Face" |
| `day` | String | Preferred day of week | "Monday" - "Friday" |
| `timeSlot` | String | Preferred time of day | "Morning" or "Afternoon" |
| `genderPref` | String | Gender preference | "Male" or "Female" |

### Output Variables (returned to Camunda Process)

The worker completes tasks with these variables:

#### Summary Variables
| Variable | Type | Description |
|----------|------|-------------|
| `topMatches` | JSON String | Array of all top matches with full details |
| `bestMatchId` | Integer | Therapist ID of best match |
| `bestMatchName` | String | Name of best match |
| `bestMatchScore` | Float | Match probability (0.0-1.0) |
| `bestMatchSpecialty` | String | Therapy type of best match |
| `totalEvaluated` | Integer | Total number of therapists evaluated |
| `matchingCompleted` | Boolean | Success flag (true) |
| `matchingTimestamp` | String | ISO 8601 timestamp |
| `workerVersion` | String | Worker version identifier |

#### Individual Match Variables (for top 3)
For matches 1-3, the following variables are set:
- `match1Id`, `match2Id`, `match3Id`: Therapist IDs
- `match1Name`, `match2Name`, `match3Name`: Therapist names
- `match1Score`, `match2Score`, `match3Score`: Match probabilities
- `match1Specialty`, `match2Specialty`, `match3Specialty`: Therapy types

### Match Result Format

The `topMatches` JSON array contains objects with this structure:

```json
[
  {
    "therapist_id": 1,
    "name": "Dr. Smith",
    "specialty": "Individual",
    "modality": "Face to Face",
    "match_probability": 0.92
  },
  {
    "therapist_id": 5,
    "name": "Dr. Patel",
    "specialty": "Individual",
    "modality": "Online",
    "match_probability": 0.87
  },
  {
    "therapist_id": 12,
    "name": "Dr. Schmidt",
    "specialty": "Couple",
    "modality": "Face to Face",
    "match_probability": 0.81
  }
]
```

## Workflow Example

### 1. Patient Request
```json
{
  "therapyType": "Individual",
  "modality": "Online",
  "day": "Wednesday",
  "timeSlot": "Morning",
  "genderPref": "Female"
}
```

### 2. Worker Processing
```
2025-12-14 12:30:15 - INFO - 🎯 Received task abc-123-def
2025-12-14 12:30:15 - INFO - 📋 Patient preferences: {'therapy_type': 'Individual', 'modality': 'Online', ...}
2025-12-14 12:30:15 - INFO - 🔍 Found 3 matches out of 20 therapists
2025-12-14 12:30:15 - INFO -   #1: Dr. Lee - Individual - Score: 98.50%
2025-12-14 12:30:15 - INFO -   #2: Dr. Kim - Individual - Score: 92.30%
2025-12-14 12:30:15 - INFO -   #3: Dr. Weber - Individual - Score: 87.10%
2025-12-14 12:30:15 - INFO - ✅ Completing task with 25 variables
2025-12-14 12:30:15 - INFO - 🏆 Best match: Dr. Lee (98.50%)
2025-12-14 12:30:15 - INFO - ✅ Task abc-123-def completed successfully
```

### 3. Result Variables Set in Process
```
bestMatchId = 2
bestMatchName = "Dr. Lee"
bestMatchScore = 0.9850
bestMatchSpecialty = "Individual"
match1Id = 2, match1Name = "Dr. Lee", match1Score = 0.9850
match2Id = 8, match2Name = "Dr. Kim", match2Score = 0.9230
match3Id = 7, match3Name = "Dr. Weber", match3Score = 0.8710
matchingCompleted = true
```

## Logging

The worker provides comprehensive logging:

### Log Levels
- **INFO**: Normal operations, task processing, matches found
- **WARNING**: Non-critical issues with individual therapists
- **ERROR**: Critical failures, connection issues, missing data

### Log Output
Logs are written to:
- **Console**: Real-time monitoring
- **File**: `therapist_worker.log` for persistent records

### Example Log Messages
```
✅ Model loaded | 20 therapists
✅ Connected to Camunda Engine
🔡 Worker subscribed to topic 'therapistMatch'
🔗 Camunda URL: https://digibp.engine.martinlab.science/engine-rest
🎯 Received task 123-abc
📋 Patient preferences: {...}
🔍 Found 3 matches out of 20 therapists
🏆 Best match: Dr. Smith (95.23%)
✅ Task 123-abc completed successfully
```

## Error Handling

The system includes robust error handling for:

### Connection Errors
- Camunda REST API connectivity check on startup
- HTTP timeout handling (10 second timeout)
- Automatic logging of connection failures

### Data Validation
- Checks for empty therapist database
- Validates model loading
- Handles missing or invalid patient preference variables

### Processing Errors
- Try-catch blocks around matching logic
- Individual therapist evaluation errors logged as warnings
- Task failures logged with full stack traces

### Graceful Shutdown
- KeyboardInterrupt handling for clean worker shutdown
- Exception logging on startup failures

## Configuration

### Environment Variables (Optional)

```python
CAMUNDA_URL = "https://digibp.engine.martinlab.science/engine-rest"
```

### Configurable Parameters

```python
TOPIC_NAME = "therapistMatch"          # External task topic
MODEL_PATH = "therapist_match_model.joblib"
THERAPISTS_PATH = "mock_therapists.joblib"
TOP_N_MATCHES = 3                      # Number of matches to return
```

## Model Files

The notebook generates and saves:

1. **`therapist_match_model.joblib`**: Trained logistic regression model
2. **`mock_therapists.joblib`**: DataFrame with 20 therapist profiles

These files are automatically loaded when the worker starts.

## Testing

### Test Camunda Connection
Run cell 3 to verify connectivity:
```
✅ Engine OK: 200
✅ External Tasks OK: 200
```

### Manual Testing
You can trigger the matching process by:
1. Starting a Camunda process instance with the required variables
2. Observing the worker logs for task pickup and completion
3. Checking the process variables for match results

## Limitations

- **Mock Data**: Uses synthetic therapist profiles (not real therapist data)
- **Static Database**: Therapist availability doesn't update dynamically
- **Binary Features**: Only considers exact matches (no fuzzy matching)
- **No Persistence**: Model and data reset on notebook restart
- **Single Tenant**: Designed for single-tenant Camunda deployment

## Future Enhancements

- [ ] Real therapist database integration
- [ ] Dynamic availability management
- [ ] Advanced matching algorithms (neural networks, ensemble methods)
- [ ] Fuzzy matching for partial criteria matches
- [ ] Patient feedback loop for model improvement
- [ ] Multi-tenant support
- [ ] Therapist profile caching
- [ ] Match explanation feature (why this therapist was selected)
- [ ] Historical match success tracking
- [ ] A/B testing framework for matching algorithms
