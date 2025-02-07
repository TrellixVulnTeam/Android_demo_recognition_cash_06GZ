# TensorFlow Android stand-alone demo

Android demo source files extracted from original TensorFlow source. (TensorFlow r0.10)

To build this demo, you don't need to prepare build environment with Bazel, and it only requires AndroidStudio.

If you would like to build jni codes, only NDK is requied to build it.

![image](http://narr.jp/private/miyoshi/tensorflow/tensorflow_screen1.png)

## How to build jni codes
First install NDK, and set path for NDK tools, and then type commands below to create .so file.

    $ cd jni-build
    $ make
    $ make install


How to train custom models with inception and docker

Prerequites

clone tensorflow recursively

enable android support in workspace
# Uncomment and update the paths in these entries to build the Android demo. Api 23 is default
android_sdk_repository(
    name = "androidsdk",
    api_level = 22,
    build_tools_version = "22.0.1",
    # Replace with path to Android SDK on your system
    path = "/media/glm/linux/",
)
#
android_ndk_repository(
    name="androidndk",
    # Replace with path to Android SDK on your system
    path="/media/glm/linux/android-ndk-r12b",
    api_level=21)

run ./configure
make
make install

install with pip....

install docker

docker run -it -v $HOME/bankovky:/last  gcr.io/tensorflow/tensorflow:latest-devel
a vysledky najdu v $HOME/bankovky diru

Je třeba updatovat model inceptiona ...
cd /tensorflow
git pull

spustit z rootu dockera
python /tensorflow/tensorflow/examples/image_retraining/retrain.py \
--bottleneck_dir=/last/bottlenecks \
--how_many_training_steps 4000 \
--model_dir=/inception \
--output_graph=/last/grafBankovky_last.pb \
--output_labels=/last/labelsBankovky_last.txt \
--image_dir /last/bankovky

deleting all chars that Android can not read
bazel-bin/tensorflow/python/tools/strip_unused 
--input_graph=inception.pb 
--output_graph=/tmp/stripped_inception.pb
--input_node_names="Mul"
--output_node_names="final_result" 
--input_binary=true

bazel-bin/tensorflow/python/tools/optimize_for_inference \
--input=/home/glm/bankovky/grafBankovky_last.pb \
--output=/home/glm/bankovky/graphBankovky_Opti.pb \
--input_names=Mul \
--output_names=final_result

Run classification if you need to check, that your graph is usable

cd /tensorflow/
bazel build tensorflow/examples/label_image:label_image
bazel-bin/tensorflow/examples/label_image/label_image \
--output_layer=final_result \
--labels=/tf_files/retrained_labels.txt \
--image=/tf_files/flower_photos/daisy/5547758_eea9edfd54_n.jpg \
--graph=/tf_files/retrained_graph.pb

jak si zkontrolovat jestli je graf funkci.Viz https://www.tensorflow.org/versions/master/how_tos/quantization/
bazel-bin/tensorflow/tools/quantization/quantize_graph \
  --input=/home/glm/bankovky/graphBankovky_Opti.pb \
  --output=/home/glm/bankovky/graphBankovky_quantizete.pb \
  --output_node_names=final_result \
  --mode=weights_rounded
bazel-bin/tensorflow/tools/quantization/quantize_graph \
  --input=/home/glm/bankovky/graphBankovky_Opti.pb \
  --output=/home/glm/bankovky/graphBankovky_quantizete.pb \
  --output_node_names=final_result \
  --mode=weights_rounded
  


Máme tu model s 87MB. Což je stále na hraniciíc sil u naších chytrých kapesní zařízení. Překopeme model, tak aby se co nejméně zatížíla RAM 
tím, že odělíme proměné(variables) od grafu, ale stále budeme mít jen jeden soubor.

bazel build tensorflow/contrib/util:convert_graphdef_memmapped_format
bazel-bin/tensorflow/contrib/util/convert_graphdef_memmapped_format \
--in_graph=/home/glm/bankovky/graphBankovky_quantizete.pb \
--out_graph=/home/glm/bankovky/graphBankovky_mapedFormat.pb  


bazel-bin/tensorflow/python/tools/freeze_graph \
--input_graph=/home/glm/PycharmProjects/untitled1/graph/def.meta \
--input_checkpoint=/home/glm/PycharmProjects/untitled1/graph/events.out.tfevents.1487338555.Glm-pc \
--output_graph=/home/glm/bazel/frozen.pb 
--output_node_names=end

If you like to create model from scratch with custom architecture, click to link described below. 
https://github.com/glmcz/Custom_model_of_Tensorflow_tutorial
