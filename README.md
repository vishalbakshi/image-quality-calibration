This synthetic image calibration experimentation UI is inspired by my experience in computer vision production pipelines, and the following quote from the [DeepFL-IQA: Weak Supervision for Deep IQA Feature Learning](https://arxiv.org/abs/2001.08113):

> We manually set the parameter values that control the distortion amount such that the perceptual visual quality of the distorted images varies linearly with the distortions parameter, from an expected rating of 1 (bad) to 5 (excellent). The distortion parameter values were chosen based on a small set of images and then applied to all images in both datasets.

This paper inspires a few questions worth exploring when you're annotating training data for an image quality model that you want to deploy into production:

- What does quality mean to a person?
- What makes something a defect?
- What does quantifying that defect from pixels actually capture?
- What does that mean for training, model outputs, and any downstream components in your machine learning pipeline?
