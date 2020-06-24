# 10fastfingers Cheater
```javascript
$(document).ready(() => {
    const maxAllowedWords = 1000;
    $("#input-row").append(`
        <button id="next" style="display:none;">Next</button>
    `);

    const input = document.querySelector('#inputfield');

    let start = 0;
    $("#next").click(function(e){
        if(maxAllowedWords <= start){
            alert("stop, Max Allowed Words Reached");
        }
        let currentWord = $("#row1").children(".highlight").text();
        e.preventDefault();
        input.value = currentWord;
        start += 1;
    });

    $(document).on("keypress", function(e){
        if(e.which == 13){
            e.preventDefault();
            $("#next").trigger("click");
        }
    });
});

```
