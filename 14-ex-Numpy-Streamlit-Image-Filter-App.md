# 14. Project Example: NumPy Image Filter App with Streamlit 🖼️

[<- Back: Data Science and Analysis Tools](./13-data-science.md) | [Next: (Conclusion or Next Steps) ->]

## Table of Contents

- [Project Concept](#project-concept)
- [Requirements](#requirements)
- [Why this Project Showcases NumPy](#why-this-project-showcases-numpy)
- [Installation](#installation)
- [Project Code](#project-code)
- [Running the App](#running-the-app)
- [Code Explanation](#code-explanation)
- [Possible Extensions](#possible-extensions)

## Project Concept

Create a simple interactive web application using Streamlit that allows users to upload an image and apply basic filters powered by NumPy array manipulations. The app will display the original image and the filtered image side-by-side.

## Requirements

1.  **Framework:** Use Streamlit for the web application interface.
2.  **Core Library:** Use NumPy for image data representation and manipulation.
3.  **Image Handling:** Use Pillow (PIL fork) to load and handle image files. Streamlit often uses this behind the scenes.
4.  **Functionality:**
    *   Allow users to upload an image file (e.g., JPG, PNG).
    *   Display the uploaded (original) image.
    *   Provide options to select different image filters (e.g., Grayscale, Brightness Adjustment, Simple Invert).
    *   Apply the selected filter using NumPy operations on the image array.
    *   Display the processed (filtered) image.
5.  **Keep it Simple:** Focus on clear examples of NumPy array operations rather than complex image processing algorithms. Aim for concise code.

## Why this Project Showcases NumPy

*   **Array Representation:** Images are naturally represented as multi-dimensional NumPy arrays (Height x Width x Color Channels).
*   **Element-wise Operations:** Filters like brightness adjustment involve simple arithmetic operations applied efficiently across the entire array.
*   **Slicing and Indexing:** Accessing color channels or specific image regions relies on NumPy's slicing capabilities.
*   **Aggregation:** Grayscaling often involves averaging pixel values across color channels (using `np.mean` or weighted averages).
*   **Data Type Handling:** Managing the `uint8` data type typical for images and clipping values to the valid 0-255 range are common NumPy tasks.

## Installation

You'll need Streamlit, NumPy, and Pillow.

```bash
pip install streamlit numpy Pillow
```

## Project Code

Save the following code as a Python file (e.g., `image_filter_app.py`).

```python
import streamlit as st
import numpy as np
from PIL import Image
import io # Used for handling byte streams for uploaded files

# --- NumPy Filter Functions ---

def apply_grayscale(img_array):
    """Converts an RGB image array to grayscale using weighted average."""
    # Ensure image is RGB (drop alpha channel if present)
    if img_array.shape[2] == 4:
        img_array = img_array[:, :, :3]

    # Weights for RGB channels (standard luminance calculation)
    weights = np.array([0.2989, 0.5870, 0.1140])

    # Calculate weighted sum using dot product for efficiency
    # We need to reshape weights to (1, 1, 3) for broadcasting
    grayscale_array = np.dot(img_array, weights.reshape(1, 1, -1)[:,:,0]).astype(np.uint8)

    # To display as an image, grayscale needs to be 3 channels (R=G=B)
    # Use np.stack to create a 3-channel image from the 1-channel grayscale
    grayscale_3channel = np.stack((grayscale_array,) * 3, axis=-1)
    return grayscale_3channel

def adjust_brightness(img_array, factor):
    """Adjusts image brightness by multiplying pixel values."""
    # Ensure factor is float for multiplication
    factor = float(factor)
    # Apply brightness adjustment and clip values to valid range [0, 255]
    # np.clip is crucial to prevent wraparound or errors
    bright_array = np.clip(img_array * factor, 0, 255).astype(np.uint8)
    return bright_array

def invert_colors(img_array):
    """Inverts the colors of the image."""
    # Subtract pixel values from 255
    inverted_array = 255 - img_array
    return inverted_array.astype(np.uint8)

# --- Streamlit App ---

st.set_page_config(layout="wide")
st.title("🎨 Simple NumPy Image Filter App")
st.write("Upload an image and apply basic filters using NumPy!")

# Sidebar for controls
st.sidebar.header("Controls")
uploaded_file = st.sidebar.file_uploader("Choose an image...", type=["jpg", "jpeg", "png"])

filter_type = st.sidebar.selectbox(
    "Select Filter",
    ["None", "Grayscale", "Invert Colors", "Brightness"]
)

# Brightness slider (conditionally shown)
brightness_factor = 1.0
if filter_type == "Brightness":
    brightness_factor = st.sidebar.slider("Brightness Factor", 0.1, 3.0, 1.0, 0.1)

# Main area for images
col1, col2 = st.columns(2) # Create two columns for side-by-side display

original_image = None
processed_image = None

if uploaded_file is not None:
    # Read the file bytes
    bytes_data = uploaded_file.getvalue()
    try:
        # Open image using PIL
        original_image = Image.open(io.BytesIO(bytes_data))

        # Convert PIL Image to NumPy array
        # Crucial step: Image data is now a NumPy array
        img_array = np.array(original_image)

        # Display original image
        with col1:
            st.header("Original Image")
            st.image(original_image, use_column_width=True)

        # Apply selected filter
        if filter_type == "Grayscale":
            processed_array = apply_grayscale(img_array.copy()) # Use copy to avoid modifying original
            processed_image = Image.fromarray(processed_array)
        elif filter_type == "Invert Colors":
            processed_array = invert_colors(img_array.copy())
            processed_image = Image.fromarray(processed_array)
        elif filter_type == "Brightness":
            processed_array = adjust_brightness(img_array.copy(), brightness_factor)
            processed_image = Image.fromarray(processed_array)
        else: # "None" or default
            processed_image = original_image # Show original if no filter selected

        # Display processed image
        with col2:
            st.header(f"Processed Image ({filter_type})")
            if processed_image:
                st.image(processed_image, use_column_width=True)
            else:
                st.write("Apply a filter to see the result.")

    except Exception as e:
        st.error(f"Error processing image: {e}")
        st.error("Please try uploading a standard image file (JPG, PNG).")

else:
    st.info("Awaiting image upload...")
```

## Running the App

1.  Save the code above as `image_filter_app.py`.
2.  Open your terminal or command prompt.
3.  Navigate to the directory where you saved the file.
4.  Run the command:

    ```bash
    streamlit run image_filter_app.py
    ```

5.  Streamlit will open the application in your web browser.

## Code Explanation

1.  **Imports:** Import `streamlit` for the UI, `numpy` for array operations, `PIL` (Pillow's `Image` module) for opening images, and `io` to handle the file buffer from Streamlit's uploader.
2.  **NumPy Filter Functions:**
    *   `apply_grayscale`: Takes a NumPy array (H x W x Channels), calculates the weighted average across the color channels (axis=2) using `np.dot` for efficiency with broadcasting, converts the result to `uint8`, and stacks it 3 times to create a displayable 3-channel grayscale image.
    *   `adjust_brightness`: Takes the array and a factor. Performs *element-wise multiplication* (`img_array * factor`). Uses `np.clip` to ensure resulting pixel values stay within the valid 0-255 range. Converts back to `uint8`.
    *   `invert_colors`: Performs *element-wise subtraction* from 255 (`255 - img_array`) to invert colors. Converts to `uint8`.
3.  **Streamlit UI:**
    *   `st.title`, `st.write`: Basic Streamlit elements for text.
    *   `st.sidebar.file_uploader`: Creates a widget in the sidebar to upload files.
    *   `st.sidebar.selectbox`/`st.sidebar.slider`: Create widgets in the sidebar to select the filter type and adjust brightness.
    *   `st.columns(2)`: Creates two columns for layout.
4.  **Image Loading and Processing:**
    *   Checks if `uploaded_file` exists.
    *   `Image.open(io.BytesIO(...))`: Opens the uploaded file bytes using PIL.
    *   `img_array = np.array(original_image)`: **This is the key conversion.** The PIL Image object is transformed into a NumPy `ndarray`. All subsequent operations work on this array.
    *   `st.image()`: Streamlit function to display images (can accept PIL Images or NumPy arrays directly in many cases).
    *   Conditional logic calls the appropriate NumPy filter function based on `filter_type`.
    *   `img_array.copy()`: Used before passing to filter functions to ensure the original array isn't modified accidentally.
    *   `Image.fromarray()`: Converts the processed NumPy array back to a PIL Image for consistent display with `st.image()`.
    *   Error handling is included for robustness.

## Possible Extensions

*   **Add More Filters:** Implement blur (e.g., simple averaging kernel using slicing), edge detection (e.g., Sobel operator using convolutions - might need SciPy or manual implementation), sepia tone.
*   **Cropping:** Allow users to select a region using coordinates or a UI element and use NumPy slicing to crop the array.
*   **Rotation/Flipping:** Use NumPy functions like `np.rot90` or slicing (`[::-1]`) to flip images.
*   **Histogram Display:** Use Matplotlib (integrated with `st.pyplot()`) to show the histogram of pixel intensities for the original and processed images.
*   **Performance:** For larger images or more complex filters, explore optimizations (vectorization is already used, but avoid explicit Python loops).

---

[<- Back: Data Science and Analysis Tools](./13-data-science.md) | [Next: (Conclusion or Next Steps) ->]
