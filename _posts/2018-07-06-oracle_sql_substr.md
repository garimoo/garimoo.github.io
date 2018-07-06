---
layout: post
title:  "[ORACLE] 파일 명으로 정렬하기(substr, instr)"
date:   2018-07-06 14:16:00
author: garimoo
categories: Database
---

밑의 테이블에서 FILE_NAME의 숫자 순으로 정렬한다고 해보자.

| FILE_NAME | FILE_ID | TABLESPACE_NAME | BYTES |
| --- | --- | --- | --- |
| /oradata/aaaaaa/bbbb/cccc_63.dbf | 1111 | cccc | 17179869184 |
| /oradata/aaaaaa/bbbb/cccc_64.dbf | 2222 | cccc | 17179869184 |

그냥 order by FILE_NAME으로 하게 된다면 *cccc_1, cccc_10, cccc_101* 이런 식으로 이상하게 정렬이 된다.
따라서 substr과 instr를 사용해서 문자열의 일부를 추출한 뒤, order by를 해 주어야 한다.

```
SELECT * FROM dba_data_files
WHERE TABLESPACE_NAME = 'cccc'
ORDER BY LENGTH(SUBSTR(FILE_NAME, INSTR(FILE_NAME,'_')),
                      SUBSTR(FILE_NAME, INSTR(FILE_NAME,'_'));
```

1. 긴 FILE_NAME에서 `INSTR`를 사용해서 `_` 가 처음으로 들어간 위치를 찾는다.
2. SUBSTR로 `_` 뒤의 문자열 추출
3. order by를 추출한 문자열의 길이 순대로 정렬한 뒤, 숫자로 정렬
