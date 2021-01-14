 @Autowired
    private RedisTemplate<String,Object> redisTemplate;

    /**
     * 初始化文章
     */
    public void initArticle(){
        String key = "article:top5";
        logger.info("从mysql中查询5个文章");
        List<Article> articles = new ArrayList<>();
        for (int i=1;i<6;i++){
            Article article = new Article();
            article.setId(Integer.toString(i));
            article.setAuthor("author"+i);
            article.setClickNum(new Random().nextInt(1000)+1);
            article.setContent("content"+i);
            article.setTitle("title"+i);
            DateTimeFormatter ftf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
            article.setCreateDate(ftf.format(LocalDateTime.ofInstant(Instant.ofEpochMilli(System.currentTimeMillis()), ZoneId.systemDefault())));
            articles.add(article);
        }
        logger.info("数据存入redis数据库中");
        redisTemplate.opsForList().rightPushAll(key,articles);

    }

    /**
     * 获取点击量排名前5的文章
     * @return 返回排名前5的文章
     */
    public List<Article> getTop5(){
        logger.info("从redis中获取前5个文章数据");
        String key = "article:top5";
        List<Object> range = redisTemplate.opsForList().range(key, 0, 4);

        assert range != null;
        return range.stream().map(object -> {
            List<Article> list = new ArrayList<>();
            if(object instanceof Article)
                list.add((Article) object);
            else
                list.addAll((List<Article>) object);
            return list;
        }).flatMap(Collection::stream).collect(Collectors.toList());
    }

    /**
     * 添加文章
     */
    public void addArticle(){
        String key="article:top5";
        logger.info("从mysql中读取数据，并添加到文章中去");
        Article article = new Article();
        article.setId("6");
        article.setAuthor("author6");
        article.setClickNum(562);
        article.setContent("content6");
        article.setTitle("title6");
        DateTimeFormatter ftf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        article.setCreateDate(ftf.format(LocalDateTime.ofInstant(Instant.ofEpochMilli(System.currentTimeMillis()), ZoneId.systemDefault())));
        redisTemplate.opsForList().rightPush(key,article);
    }

    public void orderQueue(String orderId){
        String key = "queue:"+orderId;
        redisTemplate.opsForList().leftPush(key,"1、商家发货（北京海淀区某仓库）");
        redisTemplate.opsForList().leftPush(key,"2、快递取货");
        redisTemplate.opsForList().leftPush(key,"3、北京首都机场-石家庄正定机场");
        redisTemplate.opsForList().leftPush(key,"4、正定机场到裕华区集散地");
        redisTemplate.opsForList().leftPush(key,"5、裕华区集散地到风尚水郡 ");
        redisTemplate.opsForList().leftPush(key,"6、确认收货");
    }

    public String orderTouch(String orderId){
        String keySuccess = "queue:"+orderId+":success";
        String key = "queue:"+orderId;
        return redisTemplate.opsForList().rightPopAndLeftPush(key,keySuccess).toString();
    }

    public List<String> orderSelect(String orderId){
        String key = "queue:"+orderId;
        return Objects.requireNonNull(redisTemplate.opsForList().range(key, 0, -1))
                .stream().map(Object::toString).collect(Collectors.toList());

    }

    public List<String> orderSelectSuccess(String orderId){
        String keySuccess = "queue:"+orderId+":success";
        return Objects.requireNonNull(redisTemplate.opsForList().range(keySuccess, 0, -1))
                .stream().map(Object::toString).collect(Collectors.toList());

    }
