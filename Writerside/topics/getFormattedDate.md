# getFormattedDate
* INCLER 프로젝트에서 처음으로 생성됨
* 밀리초를 XXXX.XX.XX 형태로 변환하는 함수

```Javascript
function getFormattedDate(Millidate) {
    const date = new Date(Millidate * 1000);
    const formattedDate = date.getFullYear() + "." + String(date.getMonth() + 1).padStart(2, "0") + "." + String(date.getDate()).padStart(2, "0");
    return formattedDate;
}
```