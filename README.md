# π₯ μν, κΈ°λ‘νλ€(Film Life)
## κ°μ
**μκ°** μν λ¦¬λ·°λ₯Ό μμ±νκ³  λκΈλ‘ μ¬μ©μλ€λΌλ¦¬ μν΅ν  μ μλ μ¬μ΄νΈμλλ€.<br>
**μΌμ ** 2023λ 2μ 20μΌ ~ (μ§νμ€)<br>
**μΈμ** 1μΈ κ°μΈ νλ‘μ νΈ

## μ¬μ© κΈ°μ  λ° κ°λ°νκ²½
+ Java
+ Spring Boot
+ HTML
+ CSS
+ JavaScript

## κ΅¬ν κΈ°λ₯
+ λ‘κ·ΈμΈ & νμκ°μ & νν΄

> join.js μΌλΆ

```javascript
const idNameInput = document.querySelector('#idName');
const btnJoin = document.querySelector('#btnJoin');


idNameInput.addEventListener('change', function () {
  const idName = idNameInput.value;
  axios
  .get('/checkid?idName=' + idName)
  .then(response => { displayCheckResult(response.data) })
  .catch(err => { console.log(err); })
});

function displayCheckResult(data) {
  const checkIdName = document.querySelector('#checkIdName');

  let str = '';
  if (data == 'ok') {
    str += '<div style="color:green; text-align:left;">μ¬μ©ν  μ μλ μμ΄λμλλ€.</div>';
    checkIdName.innerHTML = str;
  } else {
    str += '<div style="color:red; text-align:left;">μ¬μ©ν  μ μλ μμ΄λμλλ€.</div>';
    checkIdName.innerHTML = str;
  }
}

const passwordInput= document.querySelector('#password');
const pwChkInput = document.querySelector('#pwChk');

pwChk.addEventListener('change', function () {
  password = passwordInput.value;
  pwChk = pwChkInput.value;
  console.log(pwChk);
  console.log(password);
  const checkPw = document.querySelector('#checkPw');
  let str = '';
  if (password == pwChk) {
    str += '<div style="color:green; text-align:left;">λΉλ°λ²νΈκ° μΌμΉν©λλ€.</div>';
    checkPw.innerHTML = str;
  } else {
    str += '<div style="color:red; text-align:left;">λΉλ°λ²νΈκ° μΌμΉνμ§ μμ΅λλ€.</div>';
    checkPw.innerHTML = str;
  }
});
```

> UserController.java μΌλΆ

```java
@GetMapping("/deleteUser")
public String deleteUser(String idName) {
  userService.deleteUser(idName);
  return "redirect:/logout";
}
```

> UsersService.java μΌλΆ

```java
/**
 * νν΄μ μ¬μ© 
 * @param idName νν΄ν  μ¬μ©μ μμ΄λ 
 */
public void deleteUser(String idName) {
  Users user = userRepository.findByIdName(idName).get();
  List<ReviewScore> rs = reviewScoreRepository.findByUsersId(user.getId());
  for (ReviewScore r : rs) {
    reviewScoreRepository.delete(r);
  }

  List<Reply> replies = replyRepository.findByUsersId(user.getId());
  for (Reply r : replies) {
    replyRepository.delete(r);
  }

  List<Review> reviews = reviewRepository.findByAuthorOrderByIdDesc(idName);
  for (Review r : reviews) {
    List<Image> imageList = imageRepository.findByReviewId(r.getId());
    for (Image i : imageList) {
      imageRepository.delete(i);
    }
    reviewRepository.delete(r);
  }
  userRepository.delete(user);
}
```

---
+ λ¦¬λ·° μμ± & μμ  & μ­μ 

> ReviewController.java μΌλΆ

```java
@PostMapping("/create")
public String create(ReviewCreateDto dto) throws IOException {
  Review entity = reviewService.create(dto);

  for (MultipartFile multipartFile : dto.getFiles()) {
    imageService.saveFile(entity.getId(), multipartFile);
  }

  if (entity.getStorage() == 0) {
    return "redirect:/review/main";
  } else {
    return "redirect:/review/detail?reviewId=" + entity.getId();
  }
  
@PostMapping("/modify")
public String modify(ReviewCreateDto dto) {
  Integer reviewId = reviewService.modify(dto);
  return "redirect:/review/detail?reviewId=" + reviewId;
}

@PostMapping("/delete")
public String delete(Integer reviewId) {
  reviewService.delete(reviewId);
  return "redirect:/review/main";
}
  
```

> ReviewService.java 

```java
public Review create(ReviewCreateDto dto) {
  Review entity = reviewRepository.save(dto.toEntity());
  return entity;
}

public void delete(Integer reviewId) {
  List<ReviewScore> rs = reviewScoreRepository.findByReviewId(reviewId);
  for (ReviewScore r : rs) {
    reviewScoreRepository.deleteById(r.getId());
  }

  List<Reply> replies = replyRepository.findByReviewIdOrderByIdDesc(reviewId);
  for (Reply rp : replies) {
    replyRepository.deleteById(rp.getId());
  }

  List<Image> images = imageRepository.findByReviewId(reviewId);
  for (Image i : images) {
    imageRepository.delete(i);
  }

  reviewRepository.deleteById(reviewId);
}

@Transactional 
public Integer modify(ReviewCreateDto dto) {
  Review review = reviewRepository.findById(dto.getReviewId()).get();
  review.setStorage(1);
  review.update(dto.getTitle(), dto.getContent(), dto.getScore(), review.getStorage());
  return dto.getReviewId();
}
```


---
+ μ’μμ

> detail.html μΌλΆ

```html
<div style="display: inline-block; vertical-align: top;">
  <th:block sec:authorize="isAuthenticated()">
    <span id="loginUser" sec:authentication="principal.idName" style="display: none;"></span>
    <div th:if="${ heart } == 0">
      <button class="btnHeart" id="heart"><i class="bi bi-hearts"></i></button>
      <button class="btnHeart" id="heart-full" style="display: none;"><i class="bi bi-hearts"></i></button>
    </div>
    <div th:unless="${ heart } == 0">
      <button class="btnHeart" id="heart" style="display: none;"><i class="bi bi-hearts"></i></button>
      <button class="btnHeart" id="heart-full"><i class="bi bi-hearts"></i></button>
    </div>
  </th:block>
  <th:block sec:authorize="isAnonymous()">
    <button class="btnHeart" id="heart" onclick="loginAlert()"><i class="bi bi-hearts"></i></button>
  </th:block>
</div>
```

> detail.html μ€ <script> λΆλΆ

```html
<script>
  const reviewIdInput = document.querySelector('#reviewId').value;
  const reviewId = Number(reviewIdInput);

  const btnHeartAdd = document.querySelector('#heart');
  const btnHeartDelete = document.querySelector('#heart-full');

  let countHeart = document.querySelector('#countHeart').value;

  if (btnHeartAdd && btnHeartDelete) {
    btnHeartAdd.addEventListener('click', () => {
      const reviewId = Number(reviewIdInput);

      axios.post('/api/review/heart', null, { params: {reviewId} })
      .then(response => { 
        console.log(response.data); 
        changeFull(response.data);
      })
      .catch(error => { console.log(error); })
    })

    function changeFull(data) {
      btnHeartDelete.style.display = '';
      btnHeartAdd.style.display = 'none';
      document.querySelector('#countHeart').innerText = data;
    }

    btnHeartDelete.addEventListener('click', () => {

      axios.post('/api/review/heartDelete', null, { params: {reviewId} })
      .then(response => {
        console.log(response.data);
        changeHeart(response.data);
      })
      .catch(error => { console.log(error); })
    })

    function changeHeart(data) {
      btnHeartDelete.style.display = 'none';
      btnHeartAdd.style.display = '';
      document.querySelector('#countHeart').innerText = data;
    }
  }
</script>
```

> ReviewRestController.java μΌλΆ
  
```java
@PostMapping("/api/review/heart")
public ResponseEntity<Integer> addHeart(@AuthenticationPrincipal UserSecurityDto userSecurityDto, Integer reviewId) {
  Integer result = reviewService.addHeart(reviewId, userSecurityDto.getIdName());
  return ResponseEntity.ok(result);
}

@PostMapping("/api/review/heartDelete")
public ResponseEntity<Integer> deleteHeart(@AuthenticationPrincipal UserSecurityDto userSecurityDto, Integer reviewId) {
  Integer result = reviewService.deleteHeart(reviewId, userSecurityDto.getIdName());
  return ResponseEntity.ok(result);
}
```

---
+ λκΈ

> reply.js μΌλΆ
	
```javascript
function readAllReplies() {
  const reviewId = document.querySelector('#reviewId').value;

  axios
    .get('/api/reply/all/' + reviewId)
    .then(response => { updateReplyList(response.data) })
    .catch(err => { console.log(err) })
}

function updateReplyList(data) {
  const divReplies = document.querySelector('#replies');

  let str = '';
  for (let r of data) {
    str += '<div style="border-top: thin solid silver; padding: 20px 10px;">'
      + '<div style="font-size: small; color: gray; margin-bottom: 5px;">'
      + '<span>' + r.writer + '</span>'
      + '</div>'
      + '<div style="font-size: 15px;">' + r.replyText + '</div>'
      + '</div>'
  }
  divReplies.innerHTML = str;
}

const btnReply = document.querySelector('#btnReply');

if (btnReply) {
  btnReply.addEventListener('click', registerNewReply);

  function registerNewReply() {
    const reviewId = document.querySelector('#reviewId').value;
    const writer = document.querySelector('#writer').value;
    const replyText = document.querySelector('#replyText').value;

    if (replyText == '') {
      alert('λκΈμ μλ ₯ν΄μ£ΌμΈμ!');
      return;
    }

    const data = {
      reviewId: reviewId,
      writer: writer,
      replyText: replyText
    }

    axios.post('/api/reply', data)
      .then(response => {
        console.log(response);
        clearInputs(response.data);
        readAllReplies();
      })
      .catch(error => {
        console.log(error);
      })
  }

  function clearInputs(data) {
    document.querySelector('#replyText').value = '';
    document.querySelector('#countReply').innerText = data;
  }
```

> ReplyRestController.java μΌλΆ
	
```java
@PostMapping
public ResponseEntity<Integer> registerReply(@RequestBody ReplyRegisterDto dto) {
  Integer sizeList = replyService.create(dto);
  return ResponseEntity.ok(sizeList);
}

@GetMapping("/all/{reviewId}")
public ResponseEntity<List<ReplyReadDto>> readAllReplies(@PathVariable Integer reviewId) {
  List<ReplyReadDto> list = replyService.readReplies(reviewId);
  return ResponseEntity.ok(list);
}
```

> ReplyService.java μΌλΆ
	
```java
/**
 * Reply λ°μ΄ν°λ₯Ό DBμ λ±λ‘νλ λ©μλ
 * @param dto λ±λ‘μ νμν λ°μ΄ν°
 * @return reply PK
 */
public Integer create(ReplyRegisterDto dto) {

  Review review = reviewRepository.findById(dto.getReviewId()).get();
  Users user = usersRepository.findByIdName(dto.getWriter()).get();
  Reply reply = Reply.builder()
      .review(review).content(dto.getReplyText()).users(user).build();
  reply = replyRepository.save(reply);

  Integer sizeList = replyRepository.findByReviewIdOrderByIdDesc(dto.getReviewId()).size();

  return sizeList;
}

/**
 * ν΄λΉ λ¦¬λ·°μ λ±λ‘λ λκΈμ λΆλ¬μ¬ λ μ¬μ©νλ λ©μλ
 * @param reviewId 
 * @return
 */
public List<ReplyReadDto> readReplies(Integer reviewId) {
  List<Reply> list = replyRepository.findByReviewIdOrderByIdDesc(reviewId);
  return list.stream()
      .map(ReplyReadDto::fromEntity)
      .toList();
}
```
	
---
+ κ²μ
	
> ReviewController.java μΌλΆ
	
```java
@PostMapping("/search")
public String search(String type, String keyword, Model model) {
  List<ReviewReadDto> reviewAll = reviewService.search(type, keyword); 
  model.addAttribute("reviewAll",reviewAll);
  return "/review/main";
}
```
	
> ReviewService.java μΌλΆ
	
```java
/**
 * λ©μΈνλ©΄μμ κ²μμ μ¬μ© 
 * @param type μ΄λ€ optionμΈμ§  
 * @param keyword κ²μνκ³ μ νλ ν€μλ 
 * @return
 */
public List<ReviewReadDto> search(String type, String keyword) {
  List<ReviewReadDto> list = new ArrayList<>();

  switch(type) {
  case "r":
    List<Review> reviews = reviewRepository.findByAuthorIgnoreCaseContainingOrTitleIgnoreCaseContainingOrContentIgnoreCaseContainingOrderByIdDesc(keyword, keyword, keyword);
    for (Review r : reviews) {
      if (r.getStorage() == 1) {
        Integer[] score = countScore(r.getId());

        List<Reply> reply = replyRepository.findByReviewIdOrderByIdDesc(r.getId());
        List<Image> image = imageRepository.findByReviewId(r.getId());

        ReviewReadDto dto = ReviewReadDto.fromEntity(r, score[0], score[1], reply.size(), image.get(0).getId());
        list.add(dto);
      }
    }
    break;
  }

  return list;
}
```

---
+ μμ±κΈ λͺ¨μλ³΄κΈ°
	
> mypage.html μΌλΆ
	
```html
<div style="width:100%; margin-bottom: 15px;">
  <button id="btnChangeRelease" style="width: 49%; display: inline-block; padding: 15px 0; text-align: center; font-size: 27px; border:none; background-color:white; border-bottom: thin solid black;">λ°νν κΈ <span th:text="${ releaseSize }"></span></button>
  <button id="btnChangeSave" style="width: 49%; display: inline-block; padding: 15px 0; text-align: center; font-size: 27px; border:none; background-color:white; border-bottom: thin solid silver; color: silver;">μ μ₯λ κΈ <span th:text="${ saveSize }"></span></button>
</div>

<div id="releaseList"><!-- release list -->
  <div th:each=" reviewRelease : ${ reviewRelease }" style="margin: 10px 0; border-bottom: thin solid silver; padding: 25px;">
    <a th:href="@{ /review/detail?reviewId={reviewId} (reviewId = ${ reviewRelease.reviewId }) }">
      <div style="display: inline-block; width: 80%; vertical-align: top;">
        <div class="contents" th:text="${ reviewRelease.title }" style="font-size: 21px; padding-bottom: 5px;"></div>
        <div class="contents" th:text="${ reviewRelease.content }" style="color: silver; padding-bottom: 35px;"></div>
        <div style="font-size: small;">
          <i class="bi bi-alarm" style="margin-right: 3px;"></i><span th:text="${ #temporals.format(reviewRelease.createdTime, 'yyyy/MM/dd') }"></span>
        </div>
      </div>
      <div style="display:inline-block; width: 15%;">
        <div th:unless="${ reviewRelease.imageId } == null">
          <img th:src="|/review/images/${reviewRelease.imageId}|" style="height: 150px; obect-fit: cover;"/>
        </div>
      </div>
    </a>
  </div>
</div>

<div id="saveList" style="display: none;"><!-- save list -->
  <div th:each=" reviewSave : ${ reviewSave }" style="margin: 10px 0; border-bottom: thin solid silver; padding: 25px;">
    <a th:href="@{ /review/modify?reviewId={reviewId} (reviewId = ${ reviewSave.reviewId }) }">
      <div style="display: inline-block; width: 80%; vertical-align: top;">
        <div class="contents" th:text="${ reviewSave.title }" style="font-size: 21px; padding-bottom: 5px;"></div>
        <div class="contents" th:text="${ reviewSave.content }" style="color: silver; padding-bottom: 35px;"></div>
        <div style="font-size: small;">
          <i class="bi bi-alarm" style="margin-right: 3px;"></i><span th:text="${ #temporals.format(reviewSave.createdTime, 'yyyy/MM/dd') }"></span>
        </div>
      </div>
      <div style="display:inline-block; width: 15%;">
        <div th:unless="${ reviewSave.imageId } == null">
          <img th:src="|/review/images/${reviewSave.imageId}|" style="height: 150px; obect-fit: cover;"/>
        </div>
      </div>
    </a>
  </div>
</div>
```

> mypage.html  <script> λΆλΆ

```html
<script>
  const btnChangeRelease = document.querySelector('#btnChangeRelease');
  const btnChangeSave = document.querySelector('#btnChangeSave');

  btnChangeRelease.addEventListener('click', () => {
    document.querySelector('#releaseList').style.display = '';	
    document.querySelector('#saveList').style.display = 'none';	
    document.querySelector('#btnChangeSave').style.color = 'silver';
    document.querySelector('#btnChangeRelease').style.color = 'black';		
    document.querySelector('#btnChangeRelease').style.borderBottom = 'thin solid black';	
    document.querySelector('#btnChangeSave').style.borderBottom = 'thin solid silver';		

  })

  btnChangeSave.addEventListener('click', () => {
    document.querySelector('#saveList').style.display = '';
    document.querySelector('#releaseList').style.display = 'none';
    document.querySelector('#btnChangeSave').style.color = 'black';
    document.querySelector('#btnChangeRelease').style.color = 'silver';
    document.querySelector('#btnChangeSave').style.borderBottom = 'thin solid black';	
    document.querySelector('#btnChangeRelease').style.borderBottom = 'thin solid silver';	

  })
</script>
```

> UserController.java μΌλΆ
	
```java
@GetMapping("/mypage")
public String mypage(String idName, Model model) {
  List<Review> reviewAll = reviewService.readUser(idName);
  List<Image> imageList = imageService.readImg(idName);
  log.info("imageList size = {}", imageList);
  List<ReviewReadDto> reviewRelease = new ArrayList<>();
  List<ReviewReadDto> reviewSave = new ArrayList<>();

  for (Review r : reviewAll) {
    if (r.getStorage() == 0) {
      if (imageList != null) {
        for (Image i : imageList) {
          if (r.getId() == i.getReview().getId()) {
            ReviewReadDto dto = ReviewReadDto.fromEntity(r, null, null, null, i.getId());
            reviewSave.add(dto);
            break;
          }

          ReviewReadDto dto = ReviewReadDto.fromEntity(r, null, null, null, null);
          reviewSave.add(dto);
          break;
        }
      } else {
        ReviewReadDto dto = ReviewReadDto.fromEntity(r, null, null, null, null);
        reviewSave.add(dto);
      }
    } else {
      if (imageList != null) {
        for (Image i : imageList) {
          if (r.getId() == i.getReview().getId()) {
            ReviewReadDto dto = ReviewReadDto.fromEntity(r, null, null, null, i.getId());
            reviewRelease.add(dto);
            break;
          }

          ReviewReadDto dto = ReviewReadDto.fromEntity(r, null, null, null, null);
          reviewRelease.add(dto);
          break;
        }
      } else {
        ReviewReadDto dto = ReviewReadDto.fromEntity(r, null, null, null, null);
        reviewRelease.add(dto);
      }
    }
  }

  Integer releaseSize = reviewRelease.size();
  Integer saveSize = reviewSave.size();
  log.info("saveSize = {}", saveSize);
  model.addAttribute("releaseSize", releaseSize);
  model.addAttribute("saveSize", saveSize);
  model.addAttribute("idName", idName);
  model.addAttribute("reviewSave", reviewSave);
  model.addAttribute("reviewRelease", reviewRelease);
  return "/user/mypage";
}
```
  
+ μ¬μ§ μλ‘λ
  
> ReviewController.java μΌλΆ
  
```java
@PostMapping("/create")
public String create(ReviewCreateDto dto) throws IOException {
  Review entity = reviewService.create(dto);

  for (MultipartFile multipartFile : dto.getFiles()) {
    imageService.saveFile(entity.getId(), multipartFile);
  }

  if (entity.getStorage() == 0) {
    return "redirect:/review/main";
  } else {
    return "redirect:/review/detail?reviewId=" + entity.getId();
  }
}
```

> ImageService.java μΌλΆ
  
```java
/**
 * create.htmlμμ μ¬μ§ μ μ₯ν  λ μ¬μ© 
 * @param reviewId μΈλν€λ‘ REVEIW νμ΄λΈκ³Ό μ°κ²° 
 * @param files μλ‘λν  νμΌ 
 * @return κΈ°λ³Έν€ λ¦¬ν΄ 
 * @throws IOException
 */
public Long saveFile(Integer reviewId, MultipartFile files) throws IOException {
  if (files.isEmpty()) {
    return null;
  }

  Review review = reviewRepository.findById(reviewId).get();

  // Original File Name
  String originName = files.getOriginalFilename();

  // μ¬μ©ν  File Name
  String uuid = UUID.randomUUID().toString();

  // νμ₯μ μΆμΆ
  String extension = originName.substring(originName.lastIndexOf("."));

  // uuid + extension
  String savedName = uuid + extension;

  // νμΌ λΆλ¬μ¬ λ μ¬μ©ν  κ²½λ‘
  String savedPath = fileImageDir + savedName;

  Image image = Image.builder()
      .originName(originName)
      .fileName(savedName)
      .filePath(savedPath)
      .review(review)
      .build();

  // λ‘μ»¬μ μ μ₯
  files.transferTo(new File(savedPath));

  Image savedFile = imageRepository.save(image);

  return savedFile.getId();
}
```

## κ΅¬μ± νλ©΄
### λ©μΈ νμ΄μ§
+ λ¦¬λ·°, μμ±μλ‘ κ²μν  μ μμΌλ©° μ€μ λ©μΈ μ΄λ―Έμ§λ₯Ό λλ₯΄λ©΄ [μ¨λ€21] κ΄λ ¨ κΈ°μ¬λ‘ μ΄λ
  
![αα¦αα΅α«1](https://user-images.githubusercontent.com/113163657/224760341-9902d642-0487-4c54-88d1-c385742db10c.png)
  
+ λ¦¬λ·° 6κ° μ΄μμΌ λλ λλ³΄κΈ° λ²νΌμ ν΅ν΄ μ μ²΄ λ¦¬λ·°λ₯Ό νμΈν  μ μμ
  
![αα¦αα΅α«2](https://user-images.githubusercontent.com/113163657/224760439-897420c2-c979-41f6-abb8-bfd61610c4a2.png)

---
### λ¦¬λ·° μμΈ νμ΄μ§
  
+ λ΄μ©μ νμΈν  μ μμΌλ©° (λ‘κ·ΈμΈμ) μ’μμλ₯Ό λλ₯Ό μ μμ
  
![αα΅αα² αα΅αα¦αα΅α― αα¦αα¦αα©α―](https://user-images.githubusercontent.com/113163657/224762497-b01e3f09-7d7f-400c-916f-0fdcf73dd161.png)
  
+ λ¦¬λ·° νλ¨μ λκΈμ μμ±ν  μ μμΌλ©° μ΅κ·Ό μ¬λΌμ¨ κΈ(3κ°)λ‘ μ΄λν  μ μμ
  
![αα΅αα¦αα΅α― αα’αΊαα³α― αα΅αΎ αα?αα₯α«αα³α―](https://user-images.githubusercontent.com/113163657/224762993-45da7d20-30dc-4a79-8bbb-73d84d79fdc6.png)

---
### λ§μ΄νμ΄μ§
  
+ μ μ₯ν κΈ, λ°νν κΈμ λͺ¨μλ³Ό μ μμ
  
![αα‘αα΅αα¦αα΅αα΅ αα‘α―αα’αΌαα³α―](https://user-images.githubusercontent.com/113163657/224763275-b8afa3c1-b134-4ddc-a817-31ef96a0f271.png)
