<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>在线文件压缩</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
</head>
<body>
    <h2>选择文件进行压缩</h2>
    <input type="file" id="fileInput" multiple>
    <button onclick="compressFiles()">压缩文件</button>
    <a id="downloadLink" style="display: none;">下载 ZIP 文件</a>

    <script>
        function compressFiles() {
            const input = document.getElementById("fileInput");
            if (input.files.length === 0) {
                alert("请选择文件！");
                return;
            }

            let zip = new JSZip();
            let fileCount = input.files.length;
            let processedCount = 0;

            for (let file of input.files) {
                let reader = new FileReader();
                reader.onload = function(event) {
                    zip.file(file.name, event.target.result);
                    processedCount++;
                    if (processedCount === fileCount) {
                        zip.generateAsync({ type: "blob" }).then(function(content) {
                            let link = document.getElementById("downloadLink");
                            link.href = URL.createObjectURL(content);
                            link.download = "compressed.zip";
                            link.style.display = "inline";
                            link.textContent = "点击下载 ZIP 文件";
                        });
                    }
                };
                reader.readAsArrayBuffer(file);
            }
        }
    </script>
</body>
</html>

[back](./)
