    




  下载LOFTER我的照片书  |

package com.xf.dao.base;

import com.xf.entity.base.BaseMongoEntity;

public interface BaseMongoEntityDao<MongoEntity extends BaseMongoEntity> {
  void insert(MongoEntity en);
  MongoEntity findbyid(String id);
}

package com.xf.dao.base;

import java.lang.reflect.ParameterizedType;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;

import com.xf.entity.base.BaseMongoEntity;

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



package com.xf.entity.base;

public  abstract  class BaseMongoEntity {
 //    public abstract long getId();
    public abstract String  getId();
}

package com.xf.entity;

import java.beans.ConstructorProperties;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;

import com.xf.entity.base.BaseMongoEntity;

@Document(collection="login")
public class LoginMongoEntity extends BaseMongoEntity {
    @Id
    public String id;
    @Indexed
    public String username;
    @Indexed
    public String password;
    /*public LoginMongoEntity(String username, String password) {
        super();
        this.username = username;
        this.password = password;
    }
    
    public LoginMongoEntity(String id, String username, String password) {
        super();
        this.id = id;
        this.username = username;
        this.password = password;
    }*/
    @Override
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    
}

package com.xf.dao;

import com.xf.dao.base.BaseMongoEntityDao;
import com.xf.entity.LoginMongoEntity;

public interface  LoginDao extends BaseMongoEntityDao<LoginMongoEntity> {



    public void insert(LoginMongoEntity en);
    
    public   LoginMongoEntity findbyid(String id);
    

}

package com.xf.dao;

import org.springframework.stereotype.Repository;

import com.xf.dao.base.BaseMongoEntityDaoImpl;
import com.xf.entity.LoginMongoEntity;
@Repository("loginDao")
public class LoginDaoImpl   extends BaseMongoEntityDaoImpl<LoginMongoEntity> implements LoginDao  {

    @Override
    public void insert(LoginMongoEntity en) {
        // TODO Auto-generated method stub
       super.insert(en);
    }

    @Override
    public LoginMongoEntity  findbyid(String id) {
        return super.findbyid(id);
    }

    

    
    
}