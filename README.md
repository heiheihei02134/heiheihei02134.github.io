<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>亚马逊课程群公告生成器</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
            background: #f0f2f5;
            color: #333;
            line-height: 1.6;
            padding: 20px;
        }

        .header {
            text-align: center;
            padding: 30px 0;
            margin-bottom: 30px;
        }

        .header h1 {
            color: #232F3E;
            font-size: 2.5em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
        }

        .header p {
            color: #666;
            font-size: 1.1em;
        }

        .container {
            max-width: 1400px;
            margin: 0 auto;
            display: flex;
            gap: 30px;
            padding: 20px;
        }

        .panel {
            flex: 1;
            background: white;
            border-radius: 15px;
            padding: 25px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: transform 0.3s ease;
        }

        .panel:hover {
            transform: translateY(-5px);
        }

        .panel h2 {
            color: #232F3E;
            margin-bottom: 20px;
            padding-bottom: 10px;
            border-bottom: 2px solid #FF9900;
        }

        textarea {
            width: 100%;
            height: 500px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 14px;
            line-height: 1.6;
            resize: vertical;
            transition: border-color 0.3s ease;
            background: #fafafa;
        }

        textarea:focus {
            outline: none;
            border-color: #FF9900;
            box-shadow: 0 0 0 2px rgba(255,153,0,0.2);
        }

        .button-group {
            margin-top: 20px;
            display: flex;
            gap: 10px;
        }

        button {
            flex: 1;
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }

        .generate-btn {
            background: #FF9900;
            color: white;
        }

        .generate-btn:hover {
            background: #FF8300;
            transform: translateY(-2px);
        }

        .copy-btn {
            background: #232F3E;
            color: white;
        }

        .copy-btn:hover {
            background: #1A242F;
            transform: translateY(-2px);
        }

        .status {
            text-align: center;
            color: #666;
            margin-top: 10px;
            font-size: 14px;
        }

        @media (max-width: 768px) {
            .container {
                flex-direction: column;
            }
            
            .panel {
                margin-bottom: 20px;
            }
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .panel {
            animation: fadeIn 0.5s ease-out;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>亚马逊课程群公告生成器</h1>
        <p>快速生成标准格式的课程群公告</p>
    </div>

    <div class="container">
        <div class="panel">
            <h2>📝 输入邮件内容</h2>
            <textarea id="emailInput" placeholder="请将邮件内容粘贴到这里..."></textarea>
            <div class="button-group">
                <button class="generate-btn" onclick="processEmail()">
                    生成群公告
                </button>
            </div>
        </div>

        <div class="panel">
            <h2>📋 生成的群公告</h2>
            <textarea id="outputAnnouncement" readonly placeholder="生成的群公告将显示在这里..."></textarea>
            <div class="button-group">
                <button class="copy-btn" onclick="copyOutput()">
                    复制群公告
                </button>
            </div>
            <div id="copyStatus" class="status"></div>
        </div>
    </div>

    <script>
        const onlineTemplate = `大家好，欢迎大家参加亚马逊全球开店官方讲堂《{courseName}》线上直播课。本群为课程前期沟通及课中问题交流群，因此会在全部课程结束后第三天解散，届时小助手会给大家发送新群入群邀请，该新群主要是针对课程咨询、以及针对参加过官方讲堂的校友不定期活动的沟通群，大家可自愿选择是否加入

‼直播课程没有回放，请妥善安排好时间

课程时间：
{courseTime}

课程链接：
{courseLinks}

（登录过程中如遇到问题，请查看培训通知附件中登录指引，如仍然无法解决请联系群内小助理）
* 请提前下载培训通知邮件中附件，并进行课前预习。请不要在群中直接分享链接、二维码、小工具或者课件，如果有任何问题，请联系小助理。

★请注意： 
1. 不论使用电脑或手机端观看，都建议使用网页浏览器打开直播链接，并确保使用报名手机号登录。
2. 登录方式：填入报名手机号后收取验证码登录。
3. ‼课件可在【培训通知】邮件中附件下载
4. 本群严禁广告、暴力、恶意刷屏等内容的发送，一经发现管理员会立刻将您移出群组。`;

        const offlineTemplate = `# Group Notice #
大家好，欢迎大家参加亚马逊全球开店官方讲堂《{courseName}》线下课程，本群为课程前期沟通及课中问题交流群，因此会在课程结束后三天后解散，届时小助手会发送入群邀请给到大家，该新群主要是针对课程咨询、以及针对参加过官方讲堂的校友不定期活动的沟通群，大家可自愿选择是否加入。

课程时间：{courseTime}
上课地点：{location}

(请预留出10分钟左右用于签到和课前准备)

★请注意：
1. 进群后请修改群昵称为：报名手机号后4位 
2. 教材会在当天发放，请妥善收好您的教材，教材数量是按照报名人数准备的，一旦遗失是无法补发的。
3. 请不要在群中直接分享链接、二维码、小工具或者课件，如果有问题，请联系小助理。
4. 本群严禁广告、暴力、恶意刷屏等内容的发送，一经发现管理员会立刻将您移出群组。`;

        function extractCourseName(content) {
            const courseNameMatch = content.match(/《(.*?)》|课程名称[：:]\s*(.*?)[\n\r]/);
            return courseNameMatch ? (courseNameMatch[1] || courseNameMatch[2]) : '';
        }

        function extractTime(content) {
            const timeMatch = content.match(/(\d{4}[-/]\d{1,2}[-/]\d{1,2}.*?(?:上午|下午|晚上)?.*?\d{1,2}[:：]\d{1,2}[-~]\d{1,2}[:：]\d{1,2})/);
            return timeMatch ? timeMatch[1] : '';
        }

        function extractLinks(content) {
            let links = '';
            const lines = content.split('\n');
            for (let line of lines) {
                if (line.includes('http')) {
                    const timeMatch = line.match(/(\d{4}[-/]\d{1,2}[-/]\d{1,2}.*?\d{1,2}[:：]\d{1,2}[-~]\d{1,2}[:：]\d{1,2})/);
                    const nameMatch = line.match(/《(.*?)》|"(.*?)"|(\S+课程\S+)/);
                    const linkMatch = line.match(/(https?:\/\/[^\s]+)/);
                    
                    if (timeMatch && linkMatch) {
                        links += `${timeMatch[1]} ${nameMatch ? nameMatch[1] || nameMatch[2] || nameMatch[3] : ''}\n${linkMatch[1]}\n\n`;
                    }
                }
            }
            return links;
        }

        function extractAddress(content) {
            const addressMatch = content.match(/地[点址][：:]\s*(.*?)[\n\r]/);
            return addressMatch ? addressMatch[1] : '';
        }

        function processEmail() {
            const emailContent = document.getElementById('emailInput').value;
            const isOnline = emailContent.includes('直播') || emailContent.includes('http');
            
            const courseName = extractCourseName(emailContent);
            const courseTime = extractTime(emailContent);
            
            let announcement;
            if (isOnline) {
                const courseLinks = extractLinks(emailContent);
                announcement = onlineTemplate
                    .replace('{courseName}', courseName)
                    .replace('{courseTime}', courseTime)
                    .replace('{courseLinks}', courseLinks);
            } else {
                const location = extractAddress(emailContent);
                announcement = offlineTemplate
                    .replace('{courseName}', courseName)
                    .replace('{courseTime}', courseTime || '9:30-18:00')
                    .replace('{location}', location);
            }
            
            document.getElementById('outputAnnouncement').value = announcement;
        }

        function copyOutput() {
            const output = document.getElementById('outputAnnouncement');
            const statusDiv = document.getElementById('copyStatus');
            
            output.select();
            document.execCommand('copy');
            
            statusDiv.textContent = '✅ 群公告已复制到剪贴板！';
            setTimeout(() => {
                statusDiv.textContent = '';
            }, 2000);
        }
    </script>
</body>
</html>
