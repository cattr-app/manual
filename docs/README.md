<script>
    var userLang = navigator.language || navigator.userLanguage; 
    var href = window.location.href;
    window.location.replace(href + userLang[0] + userLang[1] + '/');
</script>
