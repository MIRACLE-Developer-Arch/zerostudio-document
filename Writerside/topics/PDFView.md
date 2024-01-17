# PDFView

<tabs>
<tab title="Shared">

````Kotlin
@Composable
expect fun PDFViewer(
    modifier: Modifier = Modifier,
    url: String
)
````
</tab>
</tabs>
<tabs>
<tab title="Android">

````Kotlin
@Composable
actual fun PDFViewer(
    modifier: Modifier,
    url: String
) {
    val context = LocalContext.current
    var pdfUri by remember { mutableStateOf<Uri?>(null) }
    var tempFile by remember { mutableStateOf<File?>(null) }
    var isLoading by remember { mutableStateOf(true) }
    var loadError by remember { mutableStateOf(false) }

    LaunchedEffect(url) {
        isLoading = true
        loadError = false
        try {
            val result = downloadPdfFromUrl(context, url)
            pdfUri = result.first
            tempFile = result.second
        } catch (e: Exception) {
            loadError = true
        } finally {
            isLoading = false
        }
    }

    DisposableEffect(Unit) {
        onDispose {
            tempFile?.delete()
        }
    }

    Column(modifier) {
        if (isLoading) {
            Text("Loading PDF...")
        } else if (loadError) {
            Text("Error loading PDF.")
        } else if (pdfUri != null) {
            AndroidView(
                factory = { context ->
                    PDFView(context, null).apply {
                        fromUri(pdfUri)
                            .enableDoubletap(true)
                            .spacing(10)
                            .load()
                    }
                },
                modifier = modifier
            )
        }
    }
}

suspend fun downloadPdfFromUrl(context: Context, pdfUrl: String): Pair<Uri, File> {
    return withContext(Dispatchers.IO) {
        val url = URL(pdfUrl)
        val connection = url.openConnection()
        val inputStream = BufferedInputStream(connection.getInputStream())
        val outputFile = File(context.cacheDir, "downloaded_pdf.pdf")
        val outputStream = FileOutputStream(outputFile)

        inputStream.copyTo(outputStream)

        inputStream.close()
        outputStream.close()

        Pair(Uri.fromFile(outputFile), outputFile)
    }
}
````
{ collapsible="true" collapsed-title="actual fun PDFViewer()" default-state="expanded" }
</tab>
<tab title="IOS">

````Kotlin
@OptIn(ExperimentalForeignApi::class)
@Composable
actual fun PDFViewer(
    modifier: Modifier,
    url: String
) {
    Column {
        if (url == "") {
            Text("Loading PDF...")
        } else {
            UIKitView(
                factory = {
                    val pdfView = PDFView().apply {
                        // URL을 NSURL 객체로 변환
                        val nsURL = NSURL.URLWithString(url)
                        // PDFDocument를 생성하고 PDFView에 할당
                        document = PDFDocument(nsURL!!)
                        // PDFView가 PDF 문서에 자동으로 맞춰지도록 설정
                        autoScales = true
                    }
                    pdfView
                },
                modifier = modifier,
                update = {},
            )
        }
    }
}
````
{ collapsible="true" collapsed-title="actual fun PDFViewer()" default-state="expanded" }
</tab>
</tabs>