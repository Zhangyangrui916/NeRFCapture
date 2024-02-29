I modify the offline mode for Splatam.
use this python script to process the data before putting it into splatam:

```python
def readDepthImage(file_path):
    with open(file_path, 'rb') as f:
        data = f.read()
    arr = np.frombuffer(data, dtype=np.float32)
    arr = arr.reshape(transformsJson['depth_map_height'], transformsJson['depth_map_width'])
    return arr  # 1 unit = 1 meter

directory = 'Z:/240223175130/'

#读取json文件
import json
with open(directory + 'transforms.json') as f:
    transformsJson = json.load(f)

os.mkdir(directory + 'rgb/')
os.mkdir(directory + 'depth/')
paths = os.listdir(directory + 'images/')
for path in paths:
    if path.endswith('.png'):
        os.rename(directory + 'images/' + path, directory + 'rgb/' + path)
    elif path.endswith('.depth'):
        depthMap = readDepthImage(directory + 'images/' + path)
        save_depth = (depthMap*65535/float(10)).astype(np.uint16)
        cv2.imwrite(directory + 'depth/' + path.replace('.depth', '.png'), save_depth)

for frame in transformsJson['frames']:
    frame['file_path'] = frame['file_path'].replace('images', 'rgb')
    frame['file_path'] += '.png'
    frame['depth_path'] = frame['depth_path'].replace('.depth.png', '.png')
    frame['depth_path'] = frame['depth_path'].replace('images', 'depth')

with open(directory + 'transforms.json', 'w') as f:
    json.dump(transformsJson, f)
```