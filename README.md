# データセットの作成
「りんごジュース」を移動するデータを 20エピソード作成。
同様に「コーヒー」、「チョコレート」も 20エピソードずつ作成。
```
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_leader_arm \
    --robot.cameras="{ camera1: {type: opencv, index_or_path: 0, width: 1280, height:  720, fps: 30, fourcc: "MJPG"}, \
                       camera2: {type: opencv, index_or_path: 2, width:  640, height:  480, fps: 30, fourcc: "MJPG"} }" \
    --display_data=true \
    --dataset.repo_id=seeed_123/grab_apple_juice_d20 \
    --dataset.push_to_hub=false \
    --dataset.num_episodes=20 \
    --dataset.episode_time_s=30 \
    --dataset.reset_time_s=3 \
    --dataset.single_task="Grab the apple juice and put it on the black hand towel."
```


# データセットのマージ
「りんごジュース」、「コーヒー」、「チョコレート」のそれぞれ 20エピソードを 1つのデータセット（60エピソード）にマージ。
```
lerobot-edit-dataset \
    --repo_id seeed_123/grab_apple_coffee_chocolate_d60 \
    --push_to_hub=false \
    --operation.type merge \
    --operation.repo_ids "['seeed_123/grab_apple_juice_d20', 'seeed_123/grab_coffee_d20', 'seeed_123/grab_chocolate_d20']"
```


# モデルのトレーニング
60エピソードのデータを使って、60,000ステップのトレーニングを実施。
```
lerobot-train \
  --policy.path=lerobot/smolvla_base \
  --policy.push_to_hub=false \
  --dataset.repo_id=seeed_123/grab_apple_coffee_chocolate_d60 \
  --batch_size=64 \
  --steps=60000 \
  --output_dir=outputs/train/my_smolvla10\
  --job_name=my_smolvla_training10 \
  --policy.device=cuda
  ```


# ポリシーの評価
トレーニングで得られたポリシーを使って、推論タスクを実行。
以下では、「りんごジュース」を移動できるか 10エピソード分のテストを実施。
同様に「コーヒー」、「チョコレート」もテスト。
```
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower_arm \
  --robot.cameras="{ camera1: {type: opencv, index_or_path: 0, width: 1280, height:  720, fps: 30, fourcc: "MJPG"}, \
                     camera2: {type: opencv, index_or_path: 2, width:  640, height:  480, fps: 30, fourcc: "MJPG"} }" \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader_arm \
  --policy.path=outputs/train/my_smolvla10/checkpoints/060000/pretrained_model \
  --policy.device=cuda \
  --dataset.single_task="Grab the apple juice and put it on the black hand towel." \
  --dataset.repo_id=seeed_123/eval_grab_apple_juice_s10_60k_test \
  --dataset.push_to_hub=false \
  --dataset.num_episodes=10 \
  --dataset.episode_time_s=60
```