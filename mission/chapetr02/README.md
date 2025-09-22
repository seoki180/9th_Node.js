1. 내가 진행중, 진행 완료한 미션을 모아서 보기
   ```sql
   SELECT S.name, M.contents, M.point, M.status
   FROM User_Missons AS  UM
   JOIN Missions AS M
   ON UM.mission_Index = M.mission_Index
   AND UM.store_Index  = M.store_Index
   JOIN Stores AS S
   ON M.store_Index = S.store_Index
   WHERE UM.user_Index = ?
   LIMIT ? OFFSET ?
   ```
2. 리뷰 작성하는 쿼리
   ```sql
   INSERT INTO Reviews(contents, stars, store_Index, user_Index,create_at)
   VALUES (?,?,?,?,?)
   ```
3. 홈화면 쿼리
   ```sql

   SELECT
       s.name   
       s.location   
       m.contents   
       m.point  
       m.status   
       COUNT(*) OVER() AS mission_count   
   FROM Missions AS m
   JOIN Stores   AS s
     ON s.store_Index = m.store_Index
    AND s.area_Index  = m.area_Index
   LEFT JOIN User_Mission AS um  
     ON um.user_Index    = ?
    AND um.mission_Index = m.mission_Index
    AND um.store_Index   = m.store_Index
    AND um.area_Index    = m.area_Index
   WHERE m.area_Index = ?
   LIMIT ? OFFSET ?
   ```
4. 마이페이지
   ```sql
   SELECT name,point,email,phone
   FROM Users
   WHERE user_Index = ?
   ```
