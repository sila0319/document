[toc]



# AuditingEntity

``` kotlin
package com.example.practice.global.entity

import jakarta.persistence.*
import org.springframework.data.annotation.CreatedDate
import org.springframework.data.annotation.LastModifiedDate
import org.springframework.data.jpa.domain.support.AuditingEntityListener
import java.io.Serializable
import java.time.LocalDateTime

@EntityListeners(value = [AuditingEntityListener::class])
@MappedSuperclass
abstract class AuditingEntity (

): AuditingEntityId() {
    @CreatedDate
    @Column(name= "create_at", nullable = false, updatable = false)
    lateinit var  createdDate: LocalDateTime
        protected set

    @LastModifiedDate
    @Column(name = "update_at")
    lateinit var modifiedDate: LocalDateTime
        protected set
}

@EntityListeners(value = [AuditingEntityListener::class])
@MappedSuperclass
abstract class AuditingEntityId : Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long? =null
        protected set
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false
        other as AuditingEntityId
        if (id != other.id) return false
        return true
    }

    override fun hashCode(): Int {
        return id?.hashCode() ?: 0
    }

}
```

* 해당 class는 모든 entity에 적용되는 entity클래스입니다.
* 정의된 내용으로는 pk로 삼을 id:Long 타입과 생성날짜 수정날짜 컬럼입니다.
* UserEntity, PostEntity 등을 만들때 AuditingEntity를 상속받으면 됩니다.
* 코틀린에서 hashCode와 equals를 재정의를 해주어야합니다.
* 저같은 경우에는 pk를 가지고 비교하는일이 많아서 AuditingEntity에 정의를 했으나 필요하다면 다른 엔티티클래스에서 재정의 해주시면 됩니다. 



# UserEntity

``` kotlin
package com.example.practice.domain.user.entity

import com.example.practice.domain.user.dto.*
import com.example.practice.global.entity.AuditingEntity
import jakarta.persistence.Column
import jakarta.persistence.Entity
import jakarta.persistence.EnumType
import jakarta.persistence.Enumerated
import jakarta.persistence.Table

enum class Role{
    USER,ADMIN,BEN,DROP
}
@Entity
@Table(name = "users")
class User (

    userid: String,
    userpw : String,
    username : String,
    role: Role

): AuditingEntity (){
    @Column(nullable = false)
    var userid : String = userid
        protected set

    @Column(nullable = false)
    var userpw : String = userpw
        protected set

    @Column(nullable = false)
    var username : String = username
        protected set

    @Enumerated(EnumType.STRING)
    var role : Role = role
        protected set

    fun updateInfo(req: UserSaveReq) {
        this.userid = req.userid
        this.userpw = req.userpw
        this.username = req.username
        this.role = req.role

    }

}

/**
 * 사용자 요청 값을 dto로 변환하여 보여주기 위함.
 * 회원생성, 회원정보변경시 사용할 dto
 * */
fun User.toUserSaveRes() : UserSaveRes {
    return UserSaveRes(
        id = this.id,
        userid = this.userid,
        userpw = this.userpw,
        username = this.username,
        role = this.role

    )
}

/**
 * 사용자 요청 값을 dto로 변환하여 보여주기 위함.
 * 사용자 모든 정보를 담아서 보여주기 위한 dto
 * */
fun User.toUserRes() : UserRes {
    return UserRes(
        id = this.id,
        userid = this.userid,
        userpw = this.userpw,
        username = this.username,
        role = this.role,
    )
}

```

* Entity클래스 생성자 부분에서 값을 받아서 body부분에서 채워주는 형식으로 진행했습니다.
  * 진행이유는 protected set을 선언을 해주기 위해서는 생성자 부분에서 필드를 선언을 하고 생성을 하는것으로는 진행할 수 없기 때문입니다. 

* AuditingEntity를 상속받아서 자동적으로 pk,create_at,update_at이 자동적으로 생성이 됩니다. 
* Entity -> dto로 변환을 하기위한 함수를 정의를 했습니다. 
* pk가 상속을 받아서 생성이 되어서 update구문을 UserEntity클래스 안에서 정의를 하였습니다. 



# User JPA

* kotlin에서 JPA를 사용할 경우에는 주의할 점이 몇가지 있습니다.

* 첫번째로는 JPA의 구현제가 Hibernate 라는 부분입니다.

  

``` kotlin
package com.example.practice.domain.user.repository

import com.example.practice.domain.user.entity.User
import org.springframework.data.domain.Page
import org.springframework.data.domain.Pageable
import org.springframework.data.jpa.repository.JpaRepository


interface UserRepository : JpaRepository<User?, Long?>{

    fun findByUserid(userid : String) : User?

    fun findById(id : Long?) : User?

    override fun findAll(pageable: Pageable) : Page<User?>


}
```

* 해당 코드는 코틀린에서 JPA를 활용한 코드입니다. 
* 구현체는 JpaRepository를 상속받으면 됩니다. 
* 이떄 주의할점은 코틀린에서는 null을 엄격하게 관리합니다. 하지만 DB의 특성상 쿼리를 했을때 조건에 만족하는 쿼리가 없을 경우 null을 리턴하게됩니다. 그러므로 null값을 받을 수 있도록 ?을 붙여야합니다.
* 만약 해당사항을 안 붙이고 코드를 작성하게 될 경우에 springboot의 런타임익셉션이 발생하게 됩니다. 
* findAll 부분은 JpaRepository의 인터페이스에서 오버라이드를 하여 재정의를 하였습니다. 



# User Service

``` kotlin
package com.example.practice.domain.user.service

import org.springframework.stereotype.Service

import com.example.practice.domain.user.repository.UserRepository
import com.example.practice.domain.user.dto.*
import com.example.practice.domain.user.entity.*
import com.example.practice.global.exception.UserNotFoundException

import org.springframework.data.domain.Page
import org.springframework.data.domain.Pageable
import org.springframework.transaction.annotation.Transactional


@Service
class UserService (

    private val userRepository : UserRepository

){

    @Transactional(readOnly = true)
    fun getAllUser(pageable : Pageable) : Page<UserRes> =
        userRepository.findAll(pageable).map{
        it?.toUserRes()
    }


    @Transactional(readOnly = true)
    fun getUser(id : Long?) : UserRes? {
        var user : User? = userRepository.findById(id) ?: throw UserNotFoundException(id)
        return user?.toUserRes()
    }
    @Transactional
    fun createUser(userSaveReq: UserSaveReq)  : UserSaveRes {

        if( userRepository.findByUserid(userSaveReq.userid) == null){
            var user : User = userSaveReq.toEntity()
            userRepository.save(user)
            return user.toUserSaveRes()
        }else {
            throw IllegalArgumentException("중복된 id 입니다.")
        }
    }
    @Transactional
    fun updateUser(userSaveReq : UserSaveReq) : UserSaveRes{
        var user: User? = userRepository.findById(userSaveReq.id) ?: throw UserNotFoundException(userSaveReq.id)
        user!!.updateInfo(userSaveReq)
        userRepository.save(user)
        return user.toUserSaveRes()
    }

    @Transactional
    fun deleteUser(id : Long,userDeleteReq: UserDeleteReq) : String{
        if(id.equals(userDeleteReq.id)){
            var user : User? = userRepository.findById(userDeleteReq.id) ?: throw UserNotFoundException(userDeleteReq.id)
            if(userDeleteReq.checkUser(user!!)) {
                userRepository.delete(user!!)
                return "회원 탈퇴 완료 되었습니다."
            }else
                return "잘못된 요청입니다. 다시 확인해 주세요"
        }else{
            return "잘못된 요청입니다. 다시 확인해 주세요"
        }
    }
}
```

