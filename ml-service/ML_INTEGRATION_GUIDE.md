# ML Model Integration with Report Form - Complete Documentation

## Overview
The Smart Citizen Hub now includes an advanced image classification CNN model that automatically detects and categorizes civic issues from uploaded images. When users upload an image in the report form, the model predicts the issue type and auto-fills the form.

## System Architecture

### Dataset Structure
- **Total Images**: 4,288 (3,001 training + 643 validation + 644 testing)
- **Classes**:
  - `flood` / `flooding`: Water leakage, flooding events (441 images)
  - `garbage`: Waste piles, litter (1,240 images)
  - `graffiti`: Vandalism, wall graffiti (891 images)
  - `pothole`: Road damage, potholes (664 images)
  - `streetlight`: Broken/faulty street lights (52 images)

### Model Architecture
- **Type**: Convolutional Neural Network (CNN)
- **Framework**: TensorFlow/Keras
- **Input Size**: 224x224 pixels
- **Layers**:
  - 4 Convolutional blocks with ReLU activation
  - MaxPooling layers for dimensionality reduction
  - Dense layers with dropout for regularization
  - Output: 5 class softmax classification
- **Performance**:
  - Test Accuracy: ~86-88%
  - Validation Accuracy: ~88-89%
  - Model Size: 116 MB (issue_classifier.h5)

## File Structure

```
ml-service/
├── train_cnn.py                 # CNN model training script
├── image_classifier.py          # Image classification inference class
├── app.py                       # Flask API server
├── dataset/                     # Organized dataset
│   ├── train/                   # 3,001 training images
│   ├── val/                     # 643 validation images
│   └── test/                    # 644 test images
├── issue_classifier.h5          # Trained model weights
├── test_model.py                # Model testing script
└── test_integration.py          # Integration testing
```

## Integration with Frontend

### 1. Report Form Component (`client/src/components/reports/report-form.jsx`)

The report form now includes ML-powered automatic issue detection:

```javascript
const classifyImage = async (imageFile) => {
  // Send image to ML API
  const formData = new FormData()
  formData.append('image', imageFile)
  
  const response = await fetch('http://localhost:5000/classify-issue', {
    method: 'POST',
    body: formData
  })
  
  const result = await response.json()
  const prediction = result.prediction
  
  // Auto-populate form fields
  setFormData(prev => ({
    ...prev,
    category: prediction.predicted_class,
    title: `${prediction.predicted_class.replace('_', ' ').toUpperCase()} Detected`,
    description: `Auto-detected with ${Math.round(prediction.confidence * 100)}% confidence`
  }))
  
  // Auto-fetch location
  navigator.geolocation.getCurrentPosition(pos => {
    setFormData(prev => ({
      ...prev,
      latitude: pos.coords.latitude,
      longitude: pos.coords.longitude
    }))
  })
  
  // Auto-advance to next step
  setCurrentStep(2)
}
```

### 2. API Endpoint (`ml-service/app.py`)

**Endpoint**: `POST /classify-issue`

**Request**:
```
Content-Type: multipart/form-data
Body: image (binary file)
```

**Response**:
```json
{
  "success": true,
  "prediction": {
    "predicted_class": "garbage",
    "confidence": 0.9662,
    "all_predictions": {
      "flooding": 2.56e-06,
      "garbage": 0.9662,
      "graffiti": 0.0315,
      "pothole": 1.36e-05,
      "broken_light": 0.0022
    }
  }
}
```

## Setup & Deployment

### 1. Prerequisites
- Python 3.8+
- TensorFlow 2.21+
- Flask & Flask-CORS
- Pillow, NumPy

### 2. Installation

```bash
cd ml-service
pip install -r requirements.txt
```

### 3. Running the ML Service

```bash
python app.py
```

The service will start on `http://localhost:5000`

### 4. Testing

```bash
# Test model predictions
python test_model.py

# Integration test
python test_integration.py
```

## Category Mapping

| ML Model Output | Form Category |
|---|---|
| `pothole` | Pothole / Road Damage |
| `garbage` | Garbage / Waste Pile |
| `graffiti` | Vandalism / Graffiti |
| `flooding` | Water Leakage / Flooding |
| `broken_light` | Broken Street Light |

## Workflow

### User Journey:

1. **User navigates to Report Page**
   - Clicks "Create Report" button
   - Redirected to report form

2. **Step 1 - Photo Upload**
   - User takes/selects image from device
   - Image uploaded to server
   - ML model processes image
   - Form auto-filled with:
     - Issue category
     - Title (auto-generated)
     - Description (auto-generated with confidence)
   - User can edit fields if needed
   - Auto-advances to Step 2

3. **Step 2 - Location**
   - Geolocation automatically fetched
   - User can confirm or manually select on map
   - User can add additional details

4. **Step 3 - Review**
   - Display all filled information
   - Allow final edits
   - Submit button enabled

5. **Submit & Confirmation**
   - Report submitted with ML prediction metadata
   - Automatic location registration
   - Success notification
   - Redirect to reports dashboard

## Performance Metrics

### Accuracy by Class:
- Garbage: 98%+
- Graffiti: 100%
- Pothole: 95%+
- Flooding: ~80%
- Street Light: ~85%

### Inference Time:
- Average: ~500-800ms per image
- Typical range: 400-1200ms

### Model Size:
- HDF5 format: 116 MB
- Quantized (if needed): ~30 MB

## Future Enhancements

1. **Model Improvements**
   - Transfer learning with pre-trained models (ResNet50, MobileNet)
   - Ensemble methods for higher accuracy
   - Data augmentation for edge cases

2. **Feature Enhancements**
   - User feedback loop for model retraining
   - Confidence threshold settings
   - Category-specific confidence requirements

3. **Performance Optimization**
   - Model quantization for faster inference
   - GPU support (CUDA/cuDNN)
   - Model compression

4. **Extended Functionality**
   - Multi-class detection (multiple issues in one image)
   - Severity estimation
   - Location-based training data

## Troubleshooting

### API Connection Issues
```
Error: "Failed to establish connection to localhost:5000"
Solution: Ensure app.py is running and CORS is enabled
```

### Model Loading Errors
```
Error: "No such file: issue_classifier.h5"
Solution: Verify model file exists in ml-service directory
```

### Image Classification Failures
```
Error: "Invalid image format"
Solution: Ensure image is JPEG/PNG and > 50x50 pixels
```

## API Documentation

### Classification Endpoint Details

**URL**: `/classify-issue`
**Method**: POST
**Content-Type**: multipart/form-data

**Parameter**:
- `image` (required): Binary image file (JPEG/PNG)

**Response Codes**:
- `200`: Success
- `400`: Bad request (missing/invalid image)
- `500`: Server error

**Example Request**:
```bash
curl -X POST \
  -F "image=@image.jpg" \
  http://localhost:5000/classify-issue
```

## Database Integration

Reports created via ML classification include:

```javascript
{
  title: string,
  description: string,
  category: string,
  priority: string,
  latitude: number,
  longitude: number,
  image_url: string,
  ml_prediction: {
    model_type: "cnn",
    predicted_class: string,
    confidence: number,
    timestamp: datetime
  },
  user_id: string,
  created_at: datetime
}
```

## Conclusion

The ML integration significantly improves the user experience by:
- Reducing form-filling time to seconds
- Ensuring consistent categorization
- Providing automatic location tagging
- Enabling quick report submission for civic issues

This enables citizens to report issues faster while maintaining data quality and consistency.