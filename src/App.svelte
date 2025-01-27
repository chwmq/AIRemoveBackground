<script>
  import { onMount } from 'svelte';
  import { AI, User } from 'aonweb';

  let file = null;
  let imageUrl = '';
  let responseMessage = '';
  let isOversized = false;
  let scale = 2;
  let faceEnhance = true;
  let resultMessage = '';
  let outputUrl = '';
  let jsonResponse = '';
  let APP_KEY = 'b1c3a132-1703-4621-9640-63f61f6cbde9'; // 请使用你自己的 app_key
  let waiting = false;

  const MAX_FILE_SIZE = 30 * 1024 * 1024;

  const onOversize = (file) => {
    responseMessage = '文件大小超过30MB，请选择一个较小的文件。';
    isOversized = true;
  };

  const afterRead = async (file) => {
    const formData = new FormData();
    formData.append('file', file);

    try {
      const res = await uploadFile(formData);
      if (res.code == 200 && res.data) {
        imageUrl = res.data;
        responseMessage = '图片上传成功！';
      } else {
        responseMessage = '图片上传失败。';
      }
    } catch (err) {
      responseMessage = '图片上传失败。';
      console.log(err);
    }
  };

  const uploadFile = async (formData) => {
    try {
      const response = await fetch('https://tmp-file.aigic.ai/api/v1/upload?expires=1800&type=image/png', {
        method: 'POST',
        body: formData
      });
      const data = await response.json();
      return data;
    } catch (err) {
      console.error('上传错误:', err);
      return { code: 500, data: null };
    }
  };

  const handleFileChange = (event) => {
    const selectedFile = event.target.files[0];
    if (selectedFile.size > MAX_FILE_SIZE) {
      onOversize(selectedFile);
    } else {
      file = selectedFile;
      afterRead(file);
    }
  };

  async function Generate() {
    try {
      if (!imageUrl) {
        resultMessage = '请提供有效的图片网址。';
        return;
      }

      waiting = true;
      resultMessage = '';
      outputUrl = '';
      jsonResponse = '';

      const options = {
        app_key: APP_KEY,
        is_async: true,
      };

      const aonet = new AI(options);
      const user_ = new User();
      let user = await user_.islogin();

      if (!user) {
        user = await user_.login();
      }

      let params = {};
      params['rembg@cjwbw'] = {
        image: imageUrl,
      };

      let res = await aonet.prediction(['rembg@cjwbw'], { ...params });

      if (res.code === 200 && res.data) {
        let taskId = res.data.task_id;
        jsonResponse = JSON.stringify(res.data, null, 2);

        let timing = setInterval(async () => {
          try {
            let taskRes = await aonet.task(taskId);
            if (taskRes.code === 200 && taskRes.data) {
              let taskData = taskRes.data;
              jsonResponse = JSON.stringify(taskData, null, 2);

              if (taskData.result) {
                resultMessage = '背景移除成功！';
                outputUrl = taskData.result;
                clearInterval(timing);
                waiting = false;
              } else if (taskData.status === 3 || taskData.status === 4) {
                let msg = (taskData.error && taskData.error) || '操作超时。';
                throw new Error(msg);
              }
            } else if (taskRes.code === 201) {
              clearInterval(timing);
              throw new Error(taskRes.message);
            }
          } catch (error) {
            console.log(error);
            if (timing) {
              clearInterval(timing);
            }
            waiting = false;
            resultMessage = '错误: ' + error.message;
          }
        }, 1000);
      } else if (res.code === 201) {
        throw new Error(res.message);
      }
    } catch (error) {
      resultMessage = '错误: ' + error.message;
      waiting = false;
      console.error(error);
    }
  }

  onMount(() => {
    console.log('Svelte 应用已挂载！');
  });
</script>

<style>
.min-h-screen {
  min-height: 100vh;
}

.footer-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
}

.top-bar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background-color: #333;
  color: white;
  padding: 10px 20px;
  font-size: 1.5em;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  z-index: 1000;
}

body {
  background-color: black;
  color: white;
}

.bg-black {
  background-color: black;
}

.bg-gray-900 {
  background-color: #1a1a1a;
}

.bg-gray-800 {
  background-color: #2a2a2a;
}

.text-white {
  color: white;
}

.text-gray-900 {
  color: #e0e0e0;
}

.text-gray-400 {
  color: #ccc;
}

.text-gray-500 {
  color: #888;
}

.text-gray-600 {
  color: #aaa;
}

.text-gray-300 {
  color: #d1d1d1;
}

.text-green-600 {
  color: #00ff00;
}

.text-red-600 {
  color: #ff0000;
}

.bg-blue-600 {
  background-color: #007bff;
}

.hover\:bg-blue-700:hover {
  background-color: #0056b3;
}

.bg-green-600 {
  background-color: #28a745;
}

.hover\:bg-green-700:hover {
  background-color: #218838;
}

.border-gray-500 {
  border-color: #444;
}

.border-gray-300 {
  border-color: #888;
}

.shadow-md {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.rounded-lg {
  border-radius: 0.75rem;
}

.overflow-hidden {
  overflow: hidden;
}

.px-4 {
  padding-left: 1rem;
  padding-right: 1rem;
}

.py-5 {
  padding-top: 1.25rem;
  padding-bottom: 1.25rem;
}

.sm\:p-6 {
  padding: 1.5rem;
}

.mb-8 {
  margin-bottom: 2rem;
}

.mb-6 {
  margin-bottom: 1.5rem;
}

.mt-6 {
  margin-top: 1.5rem;
}

.inline-flex {
  display: inline-flex;
}

.justify-center {
  justify-content: center;
}

.py-2 {
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}

.px-4 {
  padding-left: 1rem;
  padding-right: 1rem;
}

.border-transparent {
  border-color: transparent;
}

.w-full {
  width: 100%;
}

.shadow-sm {
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.text-sm {
  font-size: 0.875rem;
}

.font-medium {
  font-weight: 500;
}

.rounded-md {
  border-radius: 0.375rem;
}

.hover\:text-indigo-400:hover {
  color: #5a67d8;
}

.cursor-pointer {
  cursor: pointer;
}

.bg-indigo-500 {
  background-color: #5a67d8;
}

.hover\:bg-indigo-600:hover {
  background-color: #4c51bf;
}

.text-indigo-500 {
  color: #5a67d8;
}

.border-dashed {
  border-style: dashed;
}

.border-2 {
  border-width: 2px;
}

.h-auto {
  height: auto;
}

.max-w-full {
  max-width: 100%;
}

.rounded-md {
  border-radius: 0.375rem;
}

.shadow-md {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
</style>

<div class="top-bar">
  移除背景
</div>

<div class="min-h-screen bg-black py-12 px-4 sm:px-6 lg:px-8" style="padding-top: 60px;">
  <div class="max-w-3xl mx-auto mt-6">
    <div class="bg-gray-900 shadow-md rounded-lg overflow-hidden">
      <div class="px-4 py-5 sm:p-6">
        
        <div class="mb-6">
          <label for="file-upload" class="block text-sm font-medium text-gray-300 mb-2">
            上传照片
          </label>
          <div class="mt-1 flex justify-center px-6 pt-5 pb-6 border-2 border-gray-500 border-dashed rounded-md">
            <div class="space-y-1 text-center">
              <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48" aria-hidden="true">
                <path d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
              </svg>
              <div class="flex text-sm text-gray-400">
                <label for="file-upload" class="relative cursor-pointer bg-gray-800 rounded-md font-medium text-indigo-500 hover:text-indigo-400">
                  <span>上传文件</span>
                  <input id="file-upload" name="file-upload" type="file" class="sr-only" on:change={handleFileChange} accept="image/*">
                </label>
                <p class="pl-1 text-gray-400">或拖拽文件</p>
              </div>
              <p class="text-xs text-gray-500">
                PNG, JPG, GIF, 最大 30MB
              </p>
            </div>
          </div>
        </div>
        
        {#if isOversized}
          <p class="mt-2 text-sm text-red-600">{responseMessage}</p>
        {/if}

        {#if responseMessage && !isOversized}
          <p class="mt-2 text-sm text-green-600">{responseMessage}</p>
        {/if}

        {#if imageUrl}
          <div class="mt-6">
            <h3 class="text-lg font-medium text-white">上传图片预览：</h3>
            <img src={imageUrl} alt="Uploaded Image" class="mt-2 max-w-full h-auto rounded-lg shadow-md" />
          </div>
          <button on:click={Generate} class="mt-4 w-full inline-flex justify-center py-2 px-4 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-blue-600 hover:bg-blue-700">
            移除背景
          </button>
        {/if}

        {#if resultMessage}
          <p class="mt-4 text-sm text-gray-400">{resultMessage}</p>
        {/if}

        {#if waiting}
          <div class="mt-4 text-center">
            <div class="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-blue-600"></div>
            <p class="mt-2 text-sm text-gray-400">AI 正在移除背景，请稍候...</p>
          </div>
        {/if}

        {#if outputUrl}
          <div class="mt-6">
            <h3 class="text-lg font-medium text-white">已移除背景的图片：</h3>
            <img src={outputUrl} alt="Background Removed Image" class="mt-2 max-w-full h-auto rounded-lg shadow-md" />
            <a href={outputUrl} download class="mt-4 w-full inline-flex justify-center py-2 px-4 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-green-600 hover:bg-green-700">
              下载高分辨率图片
            </a>
          </div>
        {/if}
      </div>
    </div>
  </div>
</div>
