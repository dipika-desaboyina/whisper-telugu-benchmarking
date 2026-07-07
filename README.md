# Benchmarking Whisper on Telugu via FasterWhisper

## Core Components : 

The structure of the pipeline has four core components : 

1. Text Normalisation 

2. Dataset loading 

3. Checkpointing 

4. Benchmarking 

## **Text Normalisation :** 

Text normalisation is essential for a Telugu dataset because two Telugu characters that appear the same to the human eye may be represented differently at byte-level in Unicode; this results in WER and CER metrics recording more mismatches than there may be in actual recognition and therefore the metrics being reported inaccurately. To avoid this, we do the following normalisation steps : 

1. Unicode Normalisation : We run NFC(Normalisation Form Canonical Composition) on the Telugu text which merges all the characters which are true equivalences, i.e., that look alike and are also byte-identical. 

2. Removing special characters using a regex : We remove all the special characters like 

   - [.,:;’”()[]|] using a regex character class to make sure WER and CER do not get impacted  by special characters. 

3. Removing leading/trailing white space : We strip all the leading/trailing white space characters using a regex and built-in strip methods. 

## **Dataset Loading :** 

We will be loading multiple HuggingFace dataset objects, resampling them, and extracting specific metadata about them that will be incorporated in our final results. For this the following components are needed : 

1. Resampling helper function : We run one resampling helper function on each HF dataset object to make sure that its sampling rate is 16 KHz when it is undergoing recognition because Whisper models are trained on 16 KHz sampling rate. 

2. Dataset loaders : We load the HF dataset objects in a standardised way across datasets and return a tuple of (“audio_array”, “sentence_text”). 

3. Dataset dictionary : A dictionary object that references all the datasets and their loader functions that the benchmarking function can loop over. 

## **Checkpointing :** 

We need to make sure every line in the results is uniquely associated with an audio clip and its corresponding sentence. For this we write two utility functions : 

1. Check  for what’s already done : A helper function that ensures every new line is unique. 

2. Append results : Our main function that appends results that are coming from the benchmarking function. 

## **Benchmarking :** 

At this stage there are two components needed : 

1. Data transcription function : A function that creates audio segments via the model.transcribe() method and creates a generator object containing segments of audio arrays, language and beam size; when these generator objects are iterated over and joined we get the actual transcription. This function is also responsible for obtaining the correct time stamps for the audio transcriptions. 

2. Benchmarking function : Our main function that takes the predictions from the transcription, compares them against reference sentences and thereby computes the WER and CER metrics as a JSON object that contains the reference, prediction, normalised reference, and normalised prediction. At the end of this function we delete the model and use gc.collect() to garbage collect so that we can reclaim the memory space and continue running the next model efficiently. 
