# imSitu
Suport, annotation, evaluation, and baseline files for the imSitu dataset. 

If you use the imSitu dataset in your research, please cite our CVPR '16 paper:

```
@inproceedings{yatskar2016,
  title={Situation Recognition: Visual Semantic Role Labeling for Image Understanding},
  author={Yatskar, Mark and Zettlemoyer, Luke and Farhadi, Ali},
  booktitle={Conference on Computer Vision and Pattern Recognition},
  year={2016}
}
```
## Installation
The installation script downloads resized images for the dataset and baseline models.
```
./install.sh 
```

## Metadata: imsitu_space.json
A json file defining the metadata used for imSitu. 

Each verb in imSitu is mapped to a FrameNet frame, for example, 'clinging' is mapped to Retaining, and each of its roles is mapped to a role associated with that frame, for example, the imSitu role 'clungto' for clinging is mapped to the framenet role 'theme'. Each noun in space corresponds to a synset from Wordnet.
 
imsitu_space.json contians these mappings, as well as definitions of verbs, roles and an abstract sentence presented to turkers to help them understand the roles quickly. Furthermore, it contains definitions of nouns and glosses for those nouns that were presented to turkers.
 
```
import json

imsitu = json.load(open("imsitu_space.json"))

nouns = imsitu["nouns"]
verbs = imsitu["verbs"]

verbs["clinging"]

# {u'abstract': u'an AGENT clings to the CLUNGTO at a PLACE',
#  u'def': u'stick to',
#  u'framenet': u'Retaining',
#  u'order': [u'agent', u'clungto', u'place'],
#  u'roles': {
#   u'agent': {u'def': u'The entity doing the cling action',u'framenet': u'agent'},
#   u'clungto': {u'def': u'The entity the AGENT is clinging to',u'framenet': u'theme'},
#   u'place': {u'def': u'The location where the cling event is happening',u'framenet': u'place'}
#  }
# }

nouns["n02129165"]

#{u'def': u'large gregarious predatory feline of Africa and India having a tawny coat with a shaggy mane in the male',
# u'gloss': [u'lion', u'king of beasts', u'Panthera leo']}

```
## Standard splits: train.json, dev.json, test.json

These files contain annotations for the train/dev/test set. Each image in the imSitu dataset is annotated with three frames corresponding to one verb. Each annotation contains a noun value from Wordnet, or the empty string, indicating empty, for every role associated with the verb

```
import json
train = json.load(open("train.json"))

train['clinging_250.jpg']
#{u'frames': [{u'agent': u'n01882714', u'clungto': u'n05563770', u'place': u''},
#  {u'agent': u'n01882714', u'clungto': u'n05563770', u'place': u''},
#  {u'agent': u'n01882714', u'clungto': u'n00007846', u'place': u''}],
# u'verb': u'clinging'}

```

## Images
Images resized to 256x256 here (3.7G):

https://s3.amazonaws.com/my89-frame-annotation/public/of500_images_resized.tar

Original images can be found here (34G) :

https://s3.amazonaws.com/my89-frame-annotation/public/of500_images_resized.tar

## Evaluation

Evaluation scripts in the style presented in https://arxiv.org/pdf/1612.00901v1.pdf

(Note, this is slightly simplified from the CVPR '16. We removed the value-full measure that computes how often an entire frame output matches a references and instead just report value-all, how often the system output matches on all roles using any annotation. This was called value-any in the original paper).  

The scripts supports two style of evaluation: 1) evaluation of a top-k list output from a model and 2) evaluation of a model directly in pytorch.

### Top-k list evaluation
```
#download a file with output of the vgg baseline crf on the dev set
wget https://s3.amazonaws.com/my89-frame-annotation/public/crf.output.tar.gz
tar -xvf crf.output.tar.gz

# evaluate crf.output against dev set output
python eval.py --format file --system_output crf.output
batch 25200  out of 25200
top-1
	verb      	32.25%
	value     	24.56%
	value-all 	14.28%
top-5
	verb      	58.64%
	value     	42.68%
	value-all 	22.75%
gold verbs
	value     	65.90%
	value-all 	29.50%
summary
	mean      	36.32%

```
The script expects the system to output one prediction per verb per image in ranked order based on the system preferance. For example:

```
> head crf.output

tattooing_117.jpg	arranging	item	n11669921	tool	n05564590	place	n04105893	agent	n10787470
tattooing_117.jpg	tattooing	tool	n03816136	place	n04105893	target	n10787470	agent	n10287213
tattooing_117.jpg	mending	item	n03309808	tool	null	place	n08588294	agent	n10787470
tattooing_117.jpg	hunching	place	n04105893	surface	n02818832	agent	n10787470
tattooing_117.jpg	clinging	clungto	n10287213	place	n04105893	agent	n10787470
tattooing_117.jpg	admiring	admired	n09827683	place	n04105893	agent	n10787470
tattooing_117.jpg	spanking	tool	n05564590	place	n03679712	victim	n10787470	agent	n10287213
tattooing_117.jpg	autographing	item	n06410904	place	n04105893	agent	n10787470	receiver	n10787470
tattooing_117.jpg	instructing	place	n04105893	student	n10787470	agent	n10787470
tattooing_117.jpg	bandaging	place	n04105893	victim	n10787470	agent	n10129825
```
Each line should contain an image in the evaluation set (in this case the dev set) followed by a verb and pairs of roles and nouns corresponding to the roles for that verb. Each of these should be tab seperated. For the dev/test set, the file should contain 12700800 lines. 

To evaluate on the test set:
 
```
python eval.py --format file --system_output crf_test.output --eval_file test.json
batch 25200  out of 25200
top-1
	verb      	32.34%
	value     	24.64%
	value-all 	14.19%
top-5
	verb      	58.88%
	value     	42.76%
	value-all 	22.55%
gold verbs
	value     	65.66%
	value-all 	28.96%
summary
	mean      	36.25%
```

To evaluate on subsets of the data based on frequency criterion of verb-role-noun combinations in the training set. For example, to evaluate on images that require prediction of verb-role-noun combinations occuring between 0 and 10 times on the training set (inclusive)

```
python eval.py --format file --system_output crf.output --sparsity_max 10
evaluating images where most rare verb-role-noun in training is x , s.t. 0 <= x <= 10
total images = 8569
batch 25200  out of 25200
top-1
	verb      	19.89%
	value     	11.68%
	value-all 	2.85%
top-5
	verb      	44.00%
	value     	24.93%
	value-all 	6.16%
gold verbs
	value     	50.80%
	value-all 	9.97%
summary
	mean      	21.28%
```

### Model output evaluation

Models are evaluated by importing models through the --include flag, and providing weights and an encoder (that maps situations to machine ids).
To evaluate an existing model, for example the baseline model with resnet 101 as the base network:

```
python eval.py --format model --include baseline_crf.py --weights_file baseline_models/baseline_resnet_101 --encoding_file baseline_models/baseline_encoder --trust_encoder --batch_size 128 --image_dir resized_256/

creating model...
resnet_101
total encoding vrn : 89766, with padding in 149085 groups : 3
loading model weights...
evaluating model...
batch 197 out of 197
top-1
	verb     	36.76%
	value    	28.12%
	value-all	16.52%
top-5
	verb     	63.62%
	value    	47.03%
	value-all	25.36%
gold verbs
	value    	68.43%
	value-all	32.01%
summary 
	mean    	39.73%
```

or for images requiring rare predictions:

```
python eval.py --format model --include baseline_crf.py --weights_file baseline_models/baseline_resnet_101 --encoding_file baseline_models/baseline_encoder --trust_encoder --batch_size 128 --image_dir resized_256/ --sparsity_max 10

evaluating images where most rare verb-role-noun in training is x , s.t. 0 <= x <= 10
total images = 8569
creating model...
resnet_101
total encoding vrn : 89766, with padding in 149085 groups : 3
loading model weights...
evaluating model...
batch 197 out of 197
top-1
	verb     	23.50%
	value    	13.19%
	value-all	2.10%
top-5
	verb     	47.38%
	value    	26.15%
	value-all	4.19%
gold verbs
	value    	51.03%
	value-all	7.09%
summary 
	mean    	21.83%
```

## Baseline Models
We currently provide three baseline models trained with resnet base networks:

<table>
<tr> <td colspan="9"> Rare Predictions (<= 10 training examples) </td> </tr>
<tr> <td>  </td><td colspan="3"> top-1 </td> <td colspan="3"> top-5 </td> <td colspan="2"> gold verbs </td> </tr>
<tr> <td>resnet-101</td> <td> 23.50% </td> <td> 13.19% </td> <td> 2.10% </td> <td> 47.38% </td> <td> 26.15 </td> <td> 4.19% </td> <td> 51.03% </td> <td> 7.09% </td> <td> 21.83 % </td> </tr>
</table>  
