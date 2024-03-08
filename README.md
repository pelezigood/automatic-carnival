from flask import Flask, render_template, request, redirect, url_for
import boto3
import os

app = Flask(__name__)

# 配置 AWS 访问凭据
aws_access_key_id = 'your_access_key_id'
aws_secret_access_key = 'your_secret_access_key'
bucket_name = 'your_bucket_name'

# 设置 S3 客户端
s3_client = boto3.client('s3', aws_access_key_id=aws_access_key_id, aws_secret_access_key=aws_secret_access_key)

# 设置上传文件的目标路径
UPLOAD_FOLDER = 'uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# 定义路由，处理文件上传请求
@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        # 检查是否有文件被上传
        if 'file' not in request.files:
            return redirect(request.url)
        
        file = request.files['file']
        
        # 如果用户未选择文件，则返回上传页面
        if file.filename == '':
            return redirect(request.url)
        
        # 将上传的文件保存到本地
        if file:
            filename = file.filename
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
            file.save(filepath)
            
            # 将文件上传到 S3
            s3_client.upload_file(filepath, bucket_name, filename)
            
            # 删除本地文件
            os.remove(filepath)
            
            return '文件上传成功！'
    return render_template('upload.html')

if __name__ == '__main__':
    app.run(debug=True)
