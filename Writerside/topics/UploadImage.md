# UploadImage
* Thanks CarBon 프로젝트에서 처음으로 생성됨
* 클라이언트에서 받은 이미지를 서버에 올리는 과정에서 이미지 내부에 텍스트를 추가함

```Javascript
const sharp = require("sharp");
const path = require("path");
const { onCall, HttpsError, storage } = require("../FirebaseDataBase");
const Util = require("../Util/Util");

const fontPath = path.join(__dirname, "..", "NotoSansKR-SemiBold.ttf");

const farmersRef = Util.farmerRef;


const uploadImages = onCall({ timeoutSeconds: 100, memory: "512MiB" }, async (request) => {
    console.log(`current user request is ${JSON.stringify(request.auth)}`);
    if (!request.auth) {
        throw new HttpsError("unauthenticated", "Request is unauthenticated");
    }

    const uid = request.auth.uid;
    const userDocRef = farmersRef.doc(uid);
    const userDoc = await userDocRef.get();

    if (!userDoc.exists) {
        throw new HttpsError("unauthenticated", "Request is unauthenticated");
    }

    const displayName = userDoc.data().displayName;
    console.log(`displayName is ${JSON.stringify(displayName)}`);


    const currentFarmLand = request.data.currentFarmLand;
    const selectedDateMillis = request.data.selectedDateMillis;
    const image = request.data.image;
    const imageGeoPoint = request.data.geoPoint;
    console.log(`${JSON.stringify(imageGeoPoint)}`);


    const userFarmLandRef = userDocRef.collection("farmland").doc(`farmland${currentFarmLand}`);
    const userFarmlandSnapshot = await userFarmLandRef.get();
    const userFarmLandData = userFarmlandSnapshot.data();
    console.log(`userFarmLandData is ${JSON.stringify(userFarmLandData)}`);

    const farmAddress = userFarmLandData.farmAddress;
    // const farmAddressGeo = userFarmLandData.farmAddress_geo;
    const farmAddressGeo_lat = imageGeoPoint.lat;
    const farmAddressGeo_long = imageGeoPoint.long;
    // const wateringDate = Util.formatDate(selectedDateMillis);
    const now = new Date();
    const vietnamTime = now.getTime() + (7 * 60 * 60 * 1000);
    const wateringDate = Util.formatDate2(vietnamTime);

    const geoPointText = farmAddress ? `Lat: ${farmAddressGeo_lat}, Lng: ${farmAddressGeo_long}` : "";

    try {
        const imageBuffer = Buffer.from(image, "base64");

        const svgText = `
        <svg width="150%" height="150%">
            <defs>
                <style type="text/css">
                    @font-face {
                        font-family: "CustomFont";
                        src: url("file://${fontPath}") format("truetype");
                    }
                </style>
            </defs>
            <text x="10" y="20" font-size="30" fill="black">
                <tspan x="10" dy="0">${displayName}</tspan>
                <tspan x="10" dy="50">${farmAddress}</tspan>
                <tspan x="10" dy="75">${geoPointText}</tspan>
                <tspan x="10" dy="100">${wateringDate}</tspan>
            </text>
        </svg>`;

        const modifiedImageBuffer = await sharp(imageBuffer).composite([
            {
                input: Buffer.from(svgText),
                gravity: "northwest",
            },
        ]).toBuffer();
    
        const bucket = storage.bucket();
        const filePath = `farmers/${request.auth.uid}/farmland${currentFarmLand}/${selectedDateMillis}.jpg`;
        const file = bucket.file(filePath);
    
        await file.save(modifiedImageBuffer, { metadata: { contentType: "image/jpeg" }});

        const wateringDates = userFarmLandData.wateringDates;
        if (wateringDates && wateringDates[selectedDateMillis]) {
            wateringDates[selectedDateMillis].completed = true;

            await userFarmLandRef.update({
                [`wateringDates.${selectedDateMillis}.completed`]: true,
            });
        }
    
        return { status: "SUCCESS" };

    } catch (error) {
        console.log("error occur! details: ", error);
        throw new HttpsError("unknown", "There is some error occur, detail is : ", error);
    }
});

module.exports = {
    uploadImages,
};
```
{ collapsible="true" default-state="collapsed" }
