# getTermsPdfUrl
* INCLER 프로젝트에서 처음으로 생성됨
* 파이어베이스 스토리지에 저장된 개인정보, 이용약관 등 pdf 링크를 https링크로 변환하는 함수

```Javascript
const getTermsPdfUrl = onCall({ timeoutSeconds: 60, memory: "256MiB" }, async (request) => {
    if (!request.auth) {
        throw new HttpsError("unauthenticated", "Request is unauthenticated");
    }

    const pdfType = request.data.pdfType;
    let filePath;

    if (pdfType === "PRIVACY_POLICY") {
        filePath = "pdf 링크"; 
    } else if (pdfType === "TERMS_OF_USE") {
        filePath = "pdf 링크";
    } else {
        throw new HttpsError("invalid-argument", "INVALID PDF TYPE");
    }

    try {
        const result = await Util.converToHttpUrl(filePath);
        return result;
    } catch (error) {
        console.error("Error getting pdf file", error);
        throw new HttpsError("unkown", "Error getting pdf file", error);
    }
});
```


# convertToHttpUrl
* INCLER 프로젝트에서 처음으로 생성됨
* 클라우드 스토리지 버킷의 gs:// 링크 형식을 https:// 형식 링크로 변환하는 함수
```Javascript
async function converToHttpUrl(gsUrl) {
    if (!gsUrl.startsWith("gs://")) {
        return gsUrl;
    }
    const bucketName = "스토리지 버킷 주소";
    const filePath = gsUrl.slice(gsUrl.indexOf("/", 5) + 1);
    const storageRef = storage.bucket(bucketName).file(filePath);
    try {
        const [downloadUrl] = await storageRef.getSignedUrl({
            action: "read",
            expires: "03-17-2025", 
        });
        return downloadUrl;
    } catch (error) {
        console.error("Error converting gs:// URL to HTTP URL: ", error);
        throw new Error("Error converting gs:// URL to HTTP URL");
    }
}
```