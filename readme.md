**The code for this project was developed during my master studies at TU Munich. I have graduated since then and unfortunately I currently don't have the time to provide support for this project. I will leave the documentation and code online, but please note that they are provided as is.**

# The Data Processing Pipeline library

The **Data Processing Pipeline** (dpp) library was developed to help machine learning researchers working with large training sets of image data in general and medical imaging data in particular to easily set-up, structure, and change their pre-processing during experiments.

The Data Processing Pipeline is organized into transformations. Each transformation does exactly one thing. One transformation might clip all values to a certain range, while another might rotate an image. The functions that define a transformation actually set up a generator that reads from a Python iterator (or generator) and applies the transformation to each of its input. This means that a pipeline can be used with any other iterator and can be used like any other iterator, for example, you can iterate through all elements using a for-loop. This structure creates very readable, easily adaptable code.

The documentation is available in html form under docs/index.html.

## Installation

Clone the git repository in a directory of your choise:

    bash:/directory/of/your/choice/$ git clone https://github.com/FelixGruen/data-processing-pipeline.git

Add the directory to your Python path, either temporarily:

    bash:/directory/of/your/choice/$ export PYTHONPATH=$PYTHONPATH:/directory/of/your/choice/data-processing-pipeline/

Or permanently:

    bash:/directory/of/your/choice/$ SITEDIR=$(python -c 'import site; site._script()' --user-site)
    bash:/directory/of/your/choice/$ mkdir -p "$SITEDIR"
    bash:/directory/of/your/choice/$ echo "/directory/of/your/choice/data-processing-pipeline/" > "$SITEDIR/dpp.pth"

See if everything works:

    bash:/directory/of/your/choice/$ python -c 'import dpp; print(dpp.__version__)'
    1.0.0

If instead you see something like this:

    bash:/directory/of/your/choice/$ python -c 'import dpp; print(dpp.__version__)'
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
    ImportError: No module named dpp

Check your PYTHONPATH variable and make sure that it actually points to the directory that contains the dpp folder, not to the dpp folder itself:

    bash:/directory/of/your/choice/$ echo $PYTHONPATH
    :/directory/of/your/choice/data-processing-pipeline/


## Defining a Pre-Processing Pipeline

Defining a pipeline is dead simple. Simply list all the transformations you would like to perform on your data. Each transformation reads from the previous node in the pipeline, a Python generator, transforms the data, and then yields the transformed data. This means, that any node can be used like any other Python iterator, e.g. in a for-loop. Here is a little example:

    import dpp

    with dpp.Pipeline() as pipe:

        # Load slices
        node = dpp.reader.file_paths(train_data_dir, input_identifier=input_identifier, ground_truth_identifier=ground_truth_identifier, random=False, iterations=1)
        node = seg.medical.load_slice(node)

        # Random rotation then crop to fit
        node = seg.random_rotation(node, probability=1.0, upper_bound=180)

        # Random transformations
        node = seg.random_resize(node, [256, 256], probability=1.0, lower_bound=0.8, upper_bound=1.1, default_pixel=-100., default_label=0)
        node = seg.random_translation(node, probability=0.5, border_usage=0.5, default_border=0.25, label_of_interest=1, default_pixel=-100., default_label=0)

        # Adjust colours and labels
        node = seg.clip_img_values(node, -100., 400.)
        node = seg.fuse_labels_greater_than(node, 0.5)

        # Prepare as input
        node = seg.transpose(node)

    for (inputs, parameters) in pipe:
        # do something with the data

By convention, the data is a tuple of a list of inputs and a dictionary of parameters. For segmentation tasks the first element in the list of inputs is the actual input, while the second element is the ground truth.

See the documentation for a list of all available transformations.
