public abstract class BaseMongoEntityDaoImpl<MongoEntity extends BaseMongoEntity>
        implements BaseMongoEntityDao<MongoEntity> {
    public Class<MongoEntity> klass;
    @Autowired
    public MongoTemplate mongoTemplate;

    /*
     * public MongoTemplate getMongoTemplate() { return mongoTemplate; }
     * 
     * public void setMongoTemplate(MongoTemplate mongoTemplate) {
     * this.mongoTemplate = mongoTemplate; }
     */

    @Override
    public void insert(MongoEntity en) {
        // TODO Auto-generated method stub
        mongoTemplate.insert(en);
    }
    @SuppressWarnings("unchecked")
    public BaseMongoEntityDaoImpl() {
        super();
        ParameterizedType genericSuperclass = (ParameterizedType) getClass()
                .getGenericSuperclass();
        this.klass = (Class<MongoEntity>) genericSuperclass
                .getActualTypeArguments()[0];
    }

    
    @Override
    public MongoEntity findbyid(String username) {
        // TODO Auto-generated method stub
        
        return   mongoTemplate.findOne(
                new Query(Criteria.where("username").is(username)), klass);
    }

}