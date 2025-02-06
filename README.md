# Cloud Speech API 3 Ways Challenge Lab
In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the course to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly. When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes. To score 100% you must successfully complete all tasks within the time period!

link course
[Cloud Speech API 3 Ways Challenge Lab](https://www.cloudskillsboost.google/course_templates/700/labs/461583)


### Task 1. Create an API key
insert syntax bellow to your terminal console.
```
export API_KEY=<your_api_key>
```

### Task 2. Create synthetic speech from text using the Text-to-Speech API
1. insert syntax bellow to your terminal console.
```
sudo apt-get install -y virtualenv
python3 -m venv venv
source venv/bin/activate
```

2. create file `synthesize-text.json` with nano, vim or etc.
```
nano synthesize-text.json
```

3. paste the following into synthesize-text.json
```
{
    'input':{
        'text':'Cloud Text-to-Speech API allows developers to include
           natural-sounding, synthetic human speech as playable audio in
           their applications. The Text-to-Speech API converts text or
           Speech Synthesis Markup Language (SSML) input into audio data
           like MP3 or LINEAR16 (the encoding used in WAV files).'
    },
    'voice':{
        'languageCode':'en-gb',
        'name':'en-GB-Standard-A',
        'ssmlGender':'FEMALE'
    },
    'audioConfig':{
        'audioEncoding':'MP3'
    }
}
```

4. Call the Text-to-Speech API to synthesize with curl
```
curl -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
  -H "Content-Type: application/json; charset=utf-8" \
  -d @synthesize-text.json "https://texttospeech.googleapis.com/v1/text:synthesize" \
  > synthesize-text.txt
```

5. create file `tts_decode.py` and paste the following code into that file
```
import argparse
from base64 import decodebytes
import json

"""
Usage:
        python tts_decode.py --input "Filled in at lab start" \
        --output "synthesize-text-audio.mp3"

"""

def decode_tts_output(input_file, output_file):
    """ Decode output from Cloud Text-to-Speech.

    input_file: the response from Cloud Text-to-Speech
    output_file: the name of the audio file to create

    """

    with open(input_file) as input:
        response = json.load(input)
        audio_data = response['audioContent']

        with open(output_file, "wb") as new_file:
            new_file.write(decodebytes(audio_data.encode('utf-8')))

if __name__ == '__main__':
    parser = argparse.ArgumentParser(
        description="Decode output from Cloud Text-to-Speech",
        formatter_class=argparse.RawDescriptionHelpFormatter)
    parser.add_argument('--input',
                       help='The response from the Text-to-Speech API.',
                       required=True)
    parser.add_argument('--output',
                       help='The name of the audio file to create',
                       required=True)

    args = parser.parse_args()
    decode_tts_output(args.input, args.output)
```

6. Now, to create an audio file using the response you received from the Text-to-Speech API, run the following command from Cloud Shell:
```
python tts_decode.py --input "synthesize-text.txt" --output "synthesize-text-audio.mp3"
```

### Task 3. Perform speech to text transcription with the Cloud Speech API
1. create file `request.json` and paste the following code into that file 
```
{
  "config": {
      "encoding":"FLAC",
      "languageCode": "fr"
  },
  "audio": {
      "uri":"gs://cloud-samples-data/speech/corbeau_renard.flac"
  }
}
```

2. call it and store the result in a file named `response.json`
```
curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > response.json
```

### Task 4. Translate text with the Cloud Translation API
1. insert syntax bellow to your terminal console 
```
t_text="これは日本語です。"
from_lang="ja"
to_lang="en"
t_response="translation_response.txt"
```

2. call the response
```
response=$(curl -s -X POST \
-H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
-H "Content-Type: application/json; charset=utf-8" \
-d "{\"q\": \"$t_text\"}" \
"https://translation.googleapis.com/language/translate/v2?key=${API_KEY}&source=ja&target=en")
echo "$response" > "$t_response"
```

### Task 5. Detect a language with the Cloud Translation API
1. insert syntax bellow to your terminal console
```
d_text="Este%é%japonês."
```

2. decode it
```
decoded_text=$(python -c "import urllib.parse; print(urllib.parse.unquote('$d_text'))")
```

3. call it with curl 
```
curl -H "Authorization: Bearer "$(gcloud auth application-default print-access-token)   -H "Content-Type: application/json; charset=utf-8"   -d "{\"q\": [\"$decoded_text\"]}"  "https://translation.googleapis.com/language/translate/v2/detect?key=${API_KEY}"   -o "detection_response.txt"
```
