---
title: "Lecture18. Point Cloud Deep Learning Example (PointNet)"
date: 2023-04-28T10:51:09+09:00
draft: false
---

## Special Part - Point Cloud Deep Learning Example (PointNet)

![Untitled.png](/kr/ros2_foxy/images18/Untitled.png?height=300px)

이번 시간에는 ROS 2와 딥러닝 모델을 결합하여 Application을 구현해보는 예시를 준비했습니다. 사실 제가 딥러닝 분야에 지식은 매우 얕은지라 모델에 대한 설명은 불가합니다. PyTorch를 통해 구현된 오픈소스에 기반하여 동작하는 ROS 2 코드를 개발하는 맥락에 집중해주시면 감사하겠습니다!

{{% notice note %}}
이번 예시는 cuda, cudnn 등 GPU를 활용하는 부분이 많습니다. 사용하시는 노트북이 GPU 드라이버를 지원한다면 괜찮지만 그렇지 않은 경우 학습 시 매우 많은 시간이 요구됩니다.
{{% /notice %}}

- 이번 시간 사용할 모델은 스탠포드의 **PointNet**으로, 3D PointCloud의 feature를 추출하는 가장 대표적인 네트워크입니다.

![Untitled1.png](/kr/ros2_foxy/images18/Untitled1.png?height=200px)

reference : [PointNet: Deep Learning on Point Sets for 3D Classification and Segmentation](https://arxiv.org/abs/1612.00593)

- pointnet 오픈소스에서 제공하는 기능은 classification과 segmentation으로 이번 예시에서는 classification에 집중해 보겠습니다.

![Untitled2.png](/kr/ros2_foxy/images18/Untitled2.png?height=300px)

### Model Training

{{% notice note %}}
참고로, Anaconda와 같이 파이썬 가상환경을 사용하고 있다면 ROS 2 python 시스템과 호환이 불가할 수 있습니다. 파이썬 가상환경에 ROS 2 관련 패키지를 설치하시거나, ROS 2 python 환경에 torch 등 필요 패키지를 설치하시기 바랍니다.
{{% /notice %}}

- Model 코드를 clone하고 학습을 위해 필요한 dataset을 다운로드합니다. shapenet dataset의 일부가 다운로드됩니다.

```bash
git clone https://github.com/fxia22/pointnet.pytorch
cd pointnet.pytorch
pip install -e .

cd scripts
# build C++ code for visualization
bash build.sh
# download dataset
bash download.sh
```

{{% notice note %}}

현재 데모에서 사용하는 dataset은 약 1.1GB로 매우 적은 종류의 물체를 담고 있습니다. 단지 빠른 데모를 위해 이렇게 적용하는 점에 유의하세요!
{{% /notice %}}

- PointNet을 학습 시켜봅시다. segmentation과 classification 중 원하는 상황에 맞추어 training을 진행합니다. dataset이 약 1GB이기 때문에 epoch을 25로 하여도 시간은 오래 걸리지 않습니다.

```bash
cd utils
python train_classification.py --dataset <dataset path> --nepoch=<number epochs> --dataset_type <modelnet40 | shapenet>
python train_segmentation.py --dataset <dataset path> --nepoch=<number epochs>

# example
python3 train_segmentation.py --dataset /home/kimsooyoung/Downloads/pointnet.pytorch/shapenetcore_partanno_segmentation_benchmark_v0 --nepoch=24
python3 train_classification.py --dataset /home/kimsooyoung/Downloads/pointnet.pytorch/shapenetcore_partanno_segmentation_benchmark_v0 --nepoch=24 --dataset_type shapenet
```

- Training 결과로 cls, seg 폴더 내 pth 파일이 생성됩니다.

![Untitled3.png](/kr/ros2_foxy/images18/Untitled3.png?height=300px)

### Confirm Trained Parameters

> ROS 2 연동 전 학습된 pth 모델을 검증해봅시다. 별도의 clone은 필요 없으며 오픈 소스의 수정으로 제가 미리 준비한 코드를 사용하겠습니다.

{{% notice note %}}
해당 코드는 isl-org의 오픈소스에 기반하였습니다. ⇒ [https://github.com/isl-org/Open3D-PointNet](https://github.com/isl-org/Open3D-PointNet)
{{% /notice %}}

- 검증 프로그램을 실행하기 위한 추가 패키지를 다운로드 받습니다.

```bash
pip install torchvision
pip install open3d
pip install --upgrade numpy
```

- 제공된 검증 코드 pointnet_test.py 내부 모델 경로와 dataset 경로를 수정한 뒤, 예시 프로그램을 실행합니다.

```python
cd Open3D-PointNet
pip install -r requirements.txt

# 코드 중 아래 내용 수정
NUM_POINTS = 10000
MODEL_PATH = '/home/kimsooyoung/Downloads/pointnet.pytorch/utils/cls/cls_model_23.pth'
DATA_FOLDER = '/home/kimsooyoung/Downloads/pointnet.pytorch/shapenetcore_partanno_segmentation_benchmark_v0'

# 데모 실행
python3 pointnet_test.py
```

![Untitled4.png](/kr/ros2_foxy/images18/Untitled4.png?height=300px)

{{% notice note %}}
prediction 결과 vector와 pointcloud 데이터를 시각화해서 보여줍니다.
{{% /notice %}}

- 현재 dataset으로 구분할 수 있는 항목들은 다음과 같습니다. 계속해서 이야기하지만 dataset 자체의 한계로 검출할 수 있는 객체가 많지는 않습니다. 😅

```bash
# Problem ontology
classes_dict = {'Airplane': 0, 'Bag': 1, 'Cap': 2, 'Car': 3, 'Chair': 4,
                'Earphone': 5, 'Guitar': 6, 'Knife': 7, 'Lamp': 8, 'Laptop': 9,
                'Motorbike': 10, 'Mug': 11, 'Pistol': 12, 'Rocket': 13,
                'Skateboard': 14, 'Table': 15}
```

### PointNet ROS 2 연동

> 드디어 ROS 2와 PointNet을 연동시켜보려 합니다. 개발의 시작 전, 항상 어떠한 기능을 구현해야 하는지, 이미 개발 가능한 것을 무엇이고 새롭게 구현해야 하는 것은 무엇인지 습관적으로 구체화시키시길 바랍니다.

![Untitled5.png](/kr/ros2_foxy/images18/Untitled5.png?height=200px)

- shapenet dataset이 아닌, Gazebo로부터의 Raw PointCloud를 subscribe하여 물체 인식을 하고자 합니다. 이를 위한 Gazebo 환경은 일전의 rgbd world를 사용하겠습니다.

![Untitled6.png](/kr/ros2_foxy/images18/Untitled6.png?height=350px)

- 센서로부터의 raw Point Cloud에서 바닥을 제거하고 물체별로 clustering을 하는 작업 또한 일전 구현한 바 있습니다.

![Untitled7.png](/kr/ros2_foxy/images18/Untitled7.png?height=400px)

- 따라서 우리는 약간의 수정과 마지막 Node, Clustering된 pointcloud를 받아 어떤 물체인지 판단하는 기능만 구현하면 됩니다.

![Untitled8.png](/kr/ros2_foxy/images18/Untitled8.png?height=200px)

- 앞선 내용들을 잘 생각하면서, 예시부터 실행해봅시다.

{{% notice note %}}
예시 실행 전, `pointnet_torch/pointnet_node.py` 코드 내 pth 파일 위치를 갱신해주세요!
{{% /notice %}}

```python
class PointNetNode(Node):

    def __init__(self):
        super().__init__('pointnet_cls_node')

        self.declare_parameter('model_path', '/home/kimsooyoung/Downloads/pointnet/pointnet.pytorch/utils/cls/cls_model_23.pth')
        self.declare_parameter('num_points', 10000)
```

```python
# Terminal 1 - gazebo launch
ros2 launch rgbd_world pointnet_world.launch.py
# Terminal 2 - object spawn
ros2 run py_service_tutorial pointnet_object_wo_gravity
# Terminal 3 - clustering
ros2 run py_pcl_tutorial pointnet_cluster_pub
# Terminal 4 - torch classification
ros2 run pointnet_torch pointnet_torch
```

![Untitled9.png](/kr/ros2_foxy/images18/Untitled9.png?height=400px)

{{% notice note %}}
마지막 터미널에서 검출된 물체와 정확도를 확인할 수 있습니다.
{{% /notice %}}

- 예제 실행 시 gpu 메모리 제한에 따른 오류가 발생할 수 있습니다. 아래 명령어를 통해 gpu 점유율을 확인하며 실행을 추천드립니다.

```jsx
nvidia-smi -l 1
```

### 코드 분석

- 학습된 dataset과 3DGEMS를 통해 확보한 gazebo model 중 겹치는 것들을 먼저 확인해보았습니다.

![Untitled10.png](/kr/ros2_foxy/images18/Untitled10.png?height=400px)

- 확인된 모델들이 등장하도록 Model spawn service client의 코드를 수정합니다

```python
def send_spawn_req(self):

    # model_name = "beer"
    model_id = input("""Enter Model name Among Below List
    1.chair_1            \t2.chair_2           \t3.chair_3
    4.labtop_mac_1       \t5.labtop_mac_2      \t6.labtop_mac_3
    7.cup_green          \t8.cup_blue          \t9.cup_yellow
    10.cup_paper         \t11.lamp_table_large \t12.lamp_table_small
    13.side_table_1      \t14.side_table_3     \t15.side_table_set_1
    16.side_table_set_2  \t17.table_dining     \t18.cafe_table
    [Type your choice]: """)
```

- 다음으로 Clustering 코드를 수정해보겠습니다. Topic 이름 변경과 더불어 전처리 과정 중 Voxel grid와 RANSAC만 적용하도록 수정했습니다.

```python
    # RANSAC plane segmentation
    # Create the segmentation object
    seg = cloud_voxed.make_segmenter()
    ...

    # Extract inliers
    extracted_inliers = cloud_voxed.extract(inliers, negative=True)

    # inliers into list for ROS 2 Conversion
    color_cluster_point_list = []
    # cloud size color generator
    color = size_color_gen(extracted_inliers)

    for i, indices in enumerate(extracted_inliers):
        color_cluster_point_list.append([extracted_inliers[i][0],
                                        extracted_inliers[i][1],
                                        extracted_inliers[i][2],
                                        rgb_to_float(color)])
```

{{% notice note %}}

카메라가 한쪽 방향만을 인지하여 아래와 같이 객체가 분리되는 상황이 발생하기 때문에 알고리즘을 간소화하였습니다.
{{% /notice %}}

![Untitled11.png](/kr/ros2_foxy/images18/Untitled11.png?height=400px)

> 마지막으로 PointCloud2 topic을 통해 Inference를 실행하는 코드입니다.

- 생성자에서 Tuning된 parameter를 upload하여 classifier를 생성합니다.

```python
class PointNetNode(Node):

    def __init__(self):
        super().__init__('pointnet_cls_node')

        self.declare_parameter('model_path', '/home/kimsooyoung/Downloads/pointnet/pointnet.pytorch/utils/cls/cls_model_23.pth')
        self.declare_parameter('num_points', 10000)

        MODEL_PATH = self.get_parameter('model_path').value
        NUM_POINTS = self.get_parameter('num_points').value

        # Create the classification network from pre-trained model
        self.classifier = PointNetCls(k=len(classes_dict.items()), num_points=NUM_POINTS)
        if torch.cuda.is_available():
            self.classifier.cuda()
            self.classifier.load_state_dict(torch.load(MODEL_PATH))
        else:
            self.classifier.load_state_dict(torch.load(MODEL_PATH, map_location='cpu'))
        self.classifier.eval()
```

- callback에서는 PointCloud2 형식의 데이터를 subscribe 한 뒤 numpy.asarray로 변형하여 pytorch에게 넘길 준비를 합니다.

```python
		def obj_callback(self, msg):
		    # 1. Convert PointCloud2 message to numpy array
		    points_list = []
		    for data in pc2.read_points(msg, skip_nans=True):
		        points_list.append([data[0], data[1], data[2]])
		    points = np.asarray(points_list, dtype=np.float32)

		    # 2. numpy resample
		    m, n = points.shape
		    choice = np.random.choice(m, 10000, replace=True)
		    point_set = points[choice, :]

		    # 3. numpy to torch
		    point_set = torch.from_numpy(point_set)
```

- classifier를 통한 Interence후 검출 결과는 다시 cpu로 넘어온 뒤 콘솔 출력됩니다.

```python
		# 4. perform inference in GPU
    if self.cb_flag == True:
        points = Variable(point_set.unsqueeze(0))
        points = points.transpose(2, 1)
        if torch.cuda.is_available():
            points = points.cuda()
        pred_logsoft, _ = self.classifier(points)

        # move data back to cpu
        pred_logsoft_cpu = pred_logsoft.data.cpu().numpy().squeeze()
        pred_soft_cpu = np.exp(pred_logsoft_cpu)
        pred_class = np.argmax(pred_soft_cpu)

        self.get_logger().info(f'Your object is a [{list(classes_dict.keys())[pred_class]}]')
        self.get_logger().info('Probability is a [{:0.3}]'.format(pred_soft_cpu[pred_class]))
```

- 전체 시스템을 rqt graph로 살펴보면 다음과 같습니다. 앞서 개요로 설명했던 Node들에 유의하며 차근차근 분석해보세요!!

![Untitled12.png](/kr/ros2_foxy/images18/Untitled12.png?height=450px)
